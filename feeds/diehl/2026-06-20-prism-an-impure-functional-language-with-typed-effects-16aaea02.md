---
title: 'Prism: An Impure Functional Language With Typed Effects'
url: https://www.stephendiehl.com/posts/prism/
published: "2026-06-20T01:00:00Z"
feed: diehl
guid: https://www.stephendiehl.com/posts/prism/
---

# Prism: An Impure Functional Language With Typed Effects

This is going to be a very nerdy post so bear with me. Here is a function. Read it the way you would read any other function, and then tell me its type.

```rust
fn fib(n) =
  var a := 0
  var b := 1
  repeat(n) fn
    let t = a + b
    a := b
    b := t
  a

```

That is a mutable loop. There is a `var`, there is assignment, there is a temporary so the swap does not eat itself. It is, line for line, the `fib` you would write in Python after deciding that recursion was a young person's game.

Its type is `Int -> Int`, it is **functional but in place**. There is no effect type even though the function has effects, because the effects are not observable from outside the function. As far as anyone calling it is concerned, this function is pure. It mutates two variables in place and then, before the door closes behind it, sweeps up the evidence and leaves no fingerprints. And the compiler does it all for you. It's the code you would write in Python with types you get from OCaml and no monads.

This is [Prism](https://github.com/sdiehl/prism), a proof of concept functional compiler I've been working on for the last three years, built around modeling effects with modern types inspired by the intellectual lineage of OCaml 5, Haskell and Koka. The big idea of the last five or six years of functional programming is that effects are real, effects are fine, and the interesting question is not how to avoid them but how to put them in the type system and then optimize them until they cost nothing.

## Effects Are Interfaces

The one idea you need is the algebraic effect handler. An effect declares operations; a handler gives them meaning. Here is a producer that yields a sequence and has no idea who is listening:

```rust
effect Gen {
  ctl yield(Int) : Unit
}

fn produce(n) : !{Gen} Unit =
  if n == 0 then
    ()
  else
    yield(n)
    produce(n - 1)

```

The `!{Gen}` in the type is the function confessing, in writing, that it performs the `yield` operation and someone upstream had better deal with it. Now we hand the same producer to two different handlers:

```rust
fn total(n) =
  handle produce(n) with
    yield(v, k) => v + k(())
    return r => 0

fn count(n) =
  handle produce(n) with
    yield(v, k) => 1 + k(())
    return r => 0

```

The `k` is the continuation, the rest of the computation, reified as an ordinary value you can hold in your hand. `total` resumes it and adds; `count` resumes it and counts. A handler can ignore `k` entirely (that is an exception), call it once (that is state, or a generator), or call it many times. This last one is the move that makes algebraic effects more than sugar. Here a handler finds Pythagorean triples by resuming the *same* continuation once per candidate, which is to say it explores a whole search tree using nothing but straight-line code and a handler that says "yes, and also try the other branch":

```rust
effect Amb {
  ctl choose(Int) : Int,
  ctl reject(Unit) : Int
}

fn triple(n) : !{Amb} Int =
  let a = choose(n)
  let b = choose(n)
  let c = choose(n)
  if a > 0 && b > 0 && a <= b && a * a + b * b == c * c then
    a * 10000 + b * 100 + c
  else
    reject(())

fn solutions(n) =
  handle triple(n) with
    choose(m, k) => flatten(map(\(i) -> k(i), range(0, m)))
    reject(u, k) => Nil
    return r => Cons(r, Nil)

fn main() =
  let sols = solutions(14)
  println(length(sols))
  println(sum(sols))

```

`choose(n)` offers a value in `0..n-1` and `reject()` prunes a dead branch, and because the handler resumes `k` once for every candidate, `triple` reads like a function that just picks three numbers.

If you have used OCaml 5 this will feel familiar, except OCaml keeps its effects out of the types, so you find out about an unhandled one at runtime, in production, on a Friday. If you have used Haskell this will also feel familiar, except in Haskell you would be assembling a monad transformer stack, lifting each operation through every layer by hand, and explaining to a junior colleague that a monad is just a monoid in the category of endofunctors, what's the problem. Prism's effects are row polymorphic. They union structurally across calls. There is nothing to stack and nothing to lift, because there is no tower, only a set.

## One Trick, Five Ways

Once effects are first class, a remarkable number of ideas from the last thirty years of language design turn out to be unified under the same mechanism:

**Exceptions** are a handler that throws away the continuation. A clause marked `final ctl` discards `k`, so its body's value becomes the handler's result and the rest of the computation is simply abandoned. No `Result` threading, no `?` confetti up the call stack, just direct-style code that stops:

```rust
fn safe_grade(n) =
  handle grade(n) with
    final ctl abort(msg) => concat("invalid: ", msg)
    return r => r

```

And because an exception is just a label in the effect row, you get extensible exceptions for free. Each distinct failure is its own operation, so the row in a function's type spells out exactly which exceptions can escape it, the way `!{Gen}` spells out that it yields. There is no root `Exception` class to inherit from and no hierarchy to edit; a new exception is just a new label. They union structurally across calls, so a function that can `abort` and a function that can `timeout` compose into one whose row carries both. Handling one of them discharges its label and leaves the rest in the row, which means partial recovery is something the type system tracks rather than something you promise in a comment:

```rust
effect Abort   { ctl abort(String) : Unit }
effect Timeout { ctl timeout(Int)  : Unit }

-- fetch's row spells out both failures it can raise
fn fetch(id) : !{Abort, Timeout} String =
  if id < 0  then abort("bad id")
  if id > 99 then timeout(id)
  "ok"

-- discharge Timeout with a fallback; Abort still escapes
fn with_default(id) : !{Abort} String =
  handle fetch(id) with
    final ctl timeout(_) => "cached"
    return r => r

```

The handler peels `Timeout` off, so `with_default` is left carrying exactly `!{Abort}`, no more and no less. Java's checked exceptions wanted to be this and could not, because they were welded to the class hierarchy instead of being an open, structural set.

**Generators and streams** are a producer that performs `emit`, transformers that catch it and re-emit, and a consumer that folds. A pipeline is handlers nested around one producer, which means there is no intermediate list, by construction:

```rust
srange(1, n).smap(square).skeep(even).stake(5).ssum()

```

Stopping early, the `stake(5)`, is just a handler dropping a continuation once it has what it needs. Cancellation produces garbage, and that garbage is reclaimed at a statically known point with no collector involved, which is the good part we will come back to. The stream library was inspired by Haskell's [pipes](https://hackage.haskell.org/package/pipes) and [conduit](https://hackage.haskell.org/package/conduit).

**Lenses** are not a library anymore they are language-integrated. They are record-update paths plus the memory model. Given three nested record types, one path expression reaches arbitrarily deep and sets several fields at once:

```rust
type Vec2   = Vec2   { x: Int, y: Int }
type Player = Player { pos: Vec2, hp: Int }
type Game   = Game   { player: Player, score: Int } deriving (Lens)

let g2 = { g | player.pos.x = 30, player.hp = 95, score = 110 }

```

That rebuilds the spine of the nested record, and when the value is uniquely owned, each rebuild reuses the cell it just took apart, so a functional update compiles to a pointer write. No optic types are allocated, nothing is composed at runtime, the path is just addresses. The entire optics ecosystem, the van Laarhoven encoding, the profunctor zoo, the operators that look like a cat walked across the keyboard, all of it collapses here into one syntax rule and a memory discipline. And when you genuinely need to pass an accessor around, `deriving (Lens)` hands you `score_of` and `with_score` as ordinary functions:

```rust
let g3 = with_score(g2, 200)   -- score_of(g3) == 200

```

They are boring functions, which is the highest compliment in functional programming!

**Mutable state** is the `var` from the opening. Each `var` desugars to a private effect with `get` and `set` operations, discharged by a handler installed at the end of its block. The state never escapes, an analysis rejects any closure that would try to smuggle it out, and the enclosing function keeps its empty row. This is the loop you would write in Python with the signature you would want in Haskell and none of the `State` monad plumbing in between.

**Failure** is the most fun, because it is functional logic programming sneaking in through the effect row. An anonymous `Fail` effect makes "this expression might not produce a value" a thing the type system already knows how to talk about. `fail()` performs it, `guard(cond)` performs it when a check is false, and the consumers read like a wish list:

```rust
let port = cfg.at_map("port") ?? cfg.at_map("https") ?? 443
let off  = customer?.tier?.discount ?? 0
let bill = [item for item in sof(cart), if prices.at_map(item) > 4]

```

`??` falls back when the left side fails. `?.` chains through options and short-circuits. The comprehension guard prunes elements that fail instead of crashing. And because `var` is itself just handler sugar, an entire block can be transactional: `transact` snapshots every live variable, runs the body in a failure context, and rolls everything back if it fails, so an overdrawn account behaves as if the purchase never happened.

```rust
transact
  balance := balance - price
  guard(balance >= 0)
  balance
else
  0

```

Five features. One underlying mechanism, viewed through a five dimensional prism. And that lovely idea is the namesake!

## Modern Types

So far we haven't seen a lot of type signatures which is the point, most of the time you can write down quasi-Python-looking code and inference is decidable and predictable via the usual complete-and-easy Dunfield-Krishnaswami algorithm. You annotate only where you genuinely cross into higher-rank territory, which is rare, and the algorithm meets you exactly at that boundary. A function can demand a genuinely polymorphic argument, declared with a `forall` on the binder, and then use it at several types in one body:

```rust
fn pick(g : forall a. (a) -> a) : Int =
  if g(true) then
    g(10)
  else
    g(20)

fn main() =
  println(pick(\(x) -> x))

```

`g` is forced to be polymorphic, so `pick` may apply it to a `Bool` and an `Int` in the same breath, which is why `main` can only hand it the identity function and not, say, a number. A Damas-Milner core would have unified `a` with `Bool` on the first call and rejected the second; here the `forall` survives into the argument.

Ad-hoc polymorphism is type classes, but Lean-flavored: instances are named values you can point at, not anonymous magic conjured by global search. You write `given Ord(a)` to ask for a dictionary. When a type has more than one instance in scope you mark one `canonical`, so implicit resolution is never a coin toss, and you name the other explicitly at the call site with `using`:

```rust
instance ordDesc : Ord(Int) {
  fn cmp(x, y) = int_cmp(y, x)
}

canonical Ord(Int) = ordInt        -- two instances share the head, so name the default

sort_by_ord(xs)                    -- the canonical Ord(Int)
sort_by_ord(xs, using ordDesc)     -- this one, reversed

```

The instances you do not care about, you do not write. One `deriving` clause off a type declaration synthesizes the boring ones, and the field accessors too:

```rust
type Vec2 = Vec2 { x: Int, y: Int } deriving (Eq, Ord, Show, Lens)

with_x(v, 7)   -- a derived setter, and FBIP-reused when v is unique

```

Classes also feed pattern matching, which is the part that tends to make PL people sit up. A `pattern` can name a class method as its view, and then that one pattern deconstructs every type with an instance, dispatched by dictionary exactly like a method call:

```rust
pattern First(n) for Peek = view peek

fn head_or(x : c, d : Int) : Int given Peek(c) =
  match x of
    First(n) => n
    _ => d

```

`First(n)` matches a `Box`, a `Range`, or anything else that is `Peek`, and `head_or` is generic over all of them at once. The pattern is as polymorphic as the function around it.

Some ad-hoc polymorphism does not even need a class. `show` is type-directed: the compiler infers the static type of its argument and synthesizes a structural printer from the real constructor names, recursing into fields, with no instance to write and no runtime type dispatch (the printer is monomorphized from the static type, so it never reads a runtime tag to decide how to print):

```rust
show(42)                   -- "42"
show([1, 2, 3])            -- "[1, 2, 3]"
show((7, false))           -- "(7, false)"
show(Node(Leaf, 1, Leaf))  -- "Node(Leaf, 1, Leaf)"

```

And the third axis is the one the rest of the post is really about: the same row variable that makes effects composable makes them polymorphic. Here is a higher-order function that calls its argument twice and adds the results. The argument's effect row is a variable `e`, which is to say `twice` does not care what `f` does, only that it returns an `Int`:

```rust
fn twice(f : (Unit) -> Int ! {| e}) = f(()) + f(())

```

The `{| e}` is "this row, and whatever else." Each call site unifies `e` with the actual row of the thunk it passes, and that is the whole trick: one definition serves a pure argument, an effectful one, and an effectful one of a completely different effect, with no overloads and no wrapping.

```rust
fn pure_use() =
  twice() fn(u)
    21

```

Here `e` unifies with the empty row `{}`. No effect is performed, so no handler is needed, and the result is just `42`. Now force the same `twice` to carry an effect through:

```rust
fn tick_use() =
  handle twice(\(u) -> tick(())) with
    tick(u, k) => \(n) -> k(n)(n + 1)
    return r => \(n) -> r

```

The thunk performs `Tick`, so `e` unifies with `{Tick}`, and the surrounding `handle` discharges exactly that one label. Swap in a thunk that performs `Say` instead and `e` becomes `{Say}`; `twice` itself never changed. A handler only ever names the labels it wants, and `e` quietly carries everything else along, which is what lets these functions compose without a transformer stack to thread them through.

The same machinery, three times: one definition, abstracted over types, over dictionaries, and over effects.

## Zero-cost Abstractions

At this point the cynical hater reader (hi Hacker News!) is thinking the thing most armchair peanut gallery types think about elegant abstractions, which is: sure, but what does it actually cost. Algebraic effects have a reputation. The textbook implementation reifies your whole computation into a free monad, a tree of "here is an operation, and here is a function for what to do with its result," and then an interpreter walks that tree allocating a small cell at every single operation. It is beautiful and it is a heap allocation per `yield`.

Prism does not do that on the path that matters. The fast path is evidence passing in the Koka lineage, with my own deviations noted where they matter: instead of reifying the computation and reaching for the handler, it carries the active handler clause to each operation site as an ordinary parameter. A `do op` becomes a direct call. So we don't need a tree, or interpreter loop, or cell. The only allocation is one closure per handler when you install it, which is O(handlers), not O(operations). You can perform `yield` a billion times under a handler that was allocated exactly once.

The free monad is still in the building, because some patterns genuinely need it (a computation that escapes into a data structure, a truly multishot resumption, a masked handler). But it is the fallback, not the default, and the compiler proves which one you are in.

Now point this at the stream pipeline from earlier:

```rust
for n in srange(1, 11).smap(square).skeep(even) do
  print(n)

```

An interprocedural flow analysis works out, across function boundaries, exactly which effect evidence each producer and transformer needs, and threads it through. The whole chain lowers to a single loop with zero intermediate lists and zero per-operation cells. This is essentially Haskell-like stream fusion, but the thing Haskell achieves with rewrite rules that fire only if you hold your imports correctly and the wind is blowing precisely the right direction while Mercury is in retrograde, and what Rust achieves by monomorphizing iterator adapters into one another at the type level. Here it falls out of the effect compiler, because once `emit` is a direct call to a known clause, inlining the clause into the producer is just inlining.

The cleanup, meanwhile, is the good part promised earlier, and it is not a garbage collector. Prism uses Perceus reference counting: every heap cell is freed at a statically known point, deterministically, with no pauses and no tracing. And then the clever idea is that we have frame-limited reuse, where a cell you just deconstructed in a pattern match gets handed straight back to the constructor on the other side of the arrow, so `map` over a uniquely owned list mutates the list in place while remaining, on paper, a pure function returning a new one. The lens update compiling to a pointer write is this same machinery. So is the loop in `fib` not allocating. Purity and mutation turn out to be the same thing viewed from different ends of an ownership analysis, which is either a deep idea about the nature of computation or quite banal, and I'm not quite sure which.

## The Runtime & LLVM Backend

If Perceus reference counting sounds familiar, it should: it is the same memory discipline Lean 4 runs on, and for the same reason, a dependently typed proof assistant cannot afford GC pauses any more than a game loop can. Lean compiles to C and links a little runtime that does the counting. Prism does the structurally identical thing, except it emits LLVM IR (through `inkwell`) and, for the same program, a text MLIR module, and then hands the result to `clang` to link against one hand-written C runtime, `prism_rt.c`, the better part of two thousand lines.

That runtime is deliberately tiny. A heap cell is four-plus words, `{ refcount, tag, arity, fields... }`, and the whole file is the allocator, the `rc_inc`/ `rc_dec` pair that the compiler sprinkles through the code, the in-place reuse allocator that the FBIP pass targets, and the bignum and string primitives. There is no collector thread, no card table, no safepoints, nothing that runs when your code is not running. The reference counting is not a service the runtime provides at runtime; it is code the compiler already wrote into your program, and the C file is just where the four or five operations it calls happen to live.

It links against plain libc `malloc` by default (there is an opt-in `mimalloc` knob for benchmarking, but the whole thesis is "do not allocate," and a fast allocator only hides the allocations you failed to eliminate). A live-cell oracle asserts the heap balances to zero at exit, so "garbage free" is a property the test suite checks rather than a thing the README asserts.

## Modeled in Lean

The interpreter sits at the bottom of a proof chain whose top is Lean 4. It is a CEK machine, a souped-up cousin of the classic control/environment/continuation machine, and that machine is mirrored in a Lean 4 model ( `models/Prism.lean`) carrying a machine-checked theorem that its small-step relation is deterministic: a program steps to exactly one answer, no wiggle room. That same machine, in its Rust incarnation, then doubles as a differential oracle. Every program in the corpus runs through the interpreter and through all three native backends, LLVM, MLIR, and the C-linked binary, and the build stays red unless all four spit out byte-identical output. Determinism is proven once at the top in Lean, then mechanically enforced all the way down by making the proven-deterministic machine the thing the compiled code has to agree with.

That foundation buys the next thing, and here is where I get to indulge the sci-fi vision: once you can prove a program is deterministic and pin every input it reads, you can record a run and replay it bit-for-bit, which is the premise of a replayable, durable-execution Prism sketched for a future version, where "safe to replay after a crash" becomes a typed property the compiler checks. A program that can be rewound, resumed across a reboot, and trusted to land in exactly the same universe it left. Time travel for the low, low price of a determinism theorem.

## Runs In Browser through WASM

The same interpreter that serves as the differential oracle compiles to wasm, so there is a [playground](https://sdiehl.github.io/prism/) where you can type Prism and run it without installing anything. It runs the program in a worker, and the buttons next to it will dump the inferred type signatures (effect rows and all) and the lowered core IR, so you can watch a `var` loop or a stream pipeline turn into the thing it actually compiles to. Every example in this post is in the dropdown. Poke at the effect rows; they are the whole point. The full source, the Lean model, and the C runtime all live in the [prism repository on GitHub](https://github.com/sdiehl/prism).

## Nerd Stuff

For completeness, if you have read this far you have clearly made some very questionable life choices (hi fellow traveller!) so here's the PL nerd stuff:

- Type inference is the usual SOTA bidirectional and higher-rank, the complete-and-easy Dunfield-Krishnaswami algorithm, so rank-N polymorphism works without you annotating your way out of it.
- Typeclasses with Lean-style named instances ( `instance ordInt : Ord(Int)`), a `canonical` designation that keeps overlapping instances coherent, explicit override ( `sort_by_ord(xs, using ordRev)`), and `deriving (Eq, Ord, Show)`.
- A mostly OCaml-ish-shaped surface syntax : layout blocks, dot chains ( `xs.over(f).keep(g).sum()`), `with` sugar for continuation-passing code, string interpolation, effect row aliases.
- Deep recursion runs in constant stack, both natively (tail calls, and tail recursion modulo a constructor) and in the interpreter (it is essentially a more advanced version of the old CEK machine, so nothing in your program can blow the host stack).

And, for the genuinely afflicted, the full intellectual lineage. Obviously standing on the work of many FP giants, but the ones that are most directly inspired Prism's design are:

- The core IR is call-by-push-value (Levy, [*Call-by-Push-Value: A Functional/Imperative Synthesis*](https://scholar.google.com/scholar?q=Call-by-Push-Value+A+Functional+Imperative+Synthesis), 2001), whose split between values and computations is the thing that makes both the effect lowering and the reference counting analysis clean rather than heroic.
- The fast effect path is evidence passing (Xie and Leijen, [*Generalized Evidence Passing for Effect Handlers*](https://scholar.google.com/scholar?q=Generalized+Evidence+Passing+for+Effect+Handlers), ICFP 2021; [*Effect Handlers, Evidently*](https://scholar.google.com/scholar?q=Effect+Handlers+Evidently), ICFP 2020), the same compilation strategy Koka uses, with the free-monad encoding (Swierstra's [*Data Types a la Carte*](https://scholar.google.com/scholar?q=Data+Types+a+la+Carte), and Kiselyov's extensible effects) kept only as the fallback.
- The memory model is Perceus (Reinking, Xie, de Moura, Leijen, [*Perceus: Garbage Free Reference Counting with Reuse*](https://scholar.google.com/scholar?q=Perceus+Garbage+Free+Reference+Counting+with+Reuse), PLDI 2021) plus frame-limited reuse (Lorenzen and Leijen, [*Reference Counting with Frame-Limited Reuse*](https://scholar.google.com/scholar?q=Reference+Counting+with+Frame-Limited+Reuse), ICFP 2021) and fully-in-place programming (Lorenzen, Leijen, Swierstra, [*FP^2: Fully in-Place Functional Programming*](https://scholar.google.com/scholar?q=FP2+Fully+in-Place+Functional+Programming), ICFP 2023), which is also what turns the lens sugar into pointer writes.
- The effect rows are row polymorphism with scoped labels (Wand 1987; Leijen, [*Extensible Records with Scoped Labels*](https://scholar.google.com/scholar?q=Extensible+Records+with+Scoped+Labels), 2005; [*Type Directed Compilation of Row-Typed Algebraic Effects*](https://scholar.google.com/scholar?q=Type+Directed+Compilation+of+Row-Typed+Algebraic+Effects), POPL 2017), and handlers themselves are Plotkin and Pretnar ( [*Handlers of Algebraic Effects*](https://scholar.google.com/scholar?q=Handlers+of+Algebraic+Effects), ESOP 2009) by way of Eff and Koka.
- Pattern matching compiles to decision trees with a usefulness matrix for exhaustiveness (Maranget, [*Compiling Pattern Matching to Good Decision Trees*](https://scholar.google.com/scholar?q=Compiling+Pattern+Matching+to+Good+Decision+Trees), 2008; [*Warnings for Pattern Matching*](https://scholar.google.com/scholar?q=Warnings+for+Pattern+Matching), 2007), and the `pattern` forms are view patterns crossed with GHC's [*Pattern Synonyms*](https://scholar.google.com/scholar?q=Pattern+Synonyms+Haskell) (2016).
- The failure layer is the Verse calculus (Augustsson, Peyton Jones, Steele, Sweeney, et al., [*The Verse Calculus*](https://scholar.google.com/scholar?q=The+Verse+Calculus+A+Core+Calculus+for+Deterministic+Functional+Logic+Programming), ICFP 2023), recovered entirely from `final ctl` handlers with no new core.
- A subset of the core is mirrored in a Lean 4 model ( `models/Prism.lean`) with a machine-checked determinism theorem, and the three backends are held byte-identical by treating the interpreter as a differential oracle.

The thesis, if a toy project gets to have one, is that "purely functional" was always a slightly defensive name for a good idea. The good idea is that effects should be visible, typed, and composable. You do not need to forbid them to get that; you need to track them. And once you are tracking them honestly, the compiler has enough information to make them free, which is the part the purity narrative never promised you, because it was too busy not making eye contact with the IO monad.

This compiler is essentially my love letter to the old 2010-20s era of functional programming which was a much simpler and happier time (or maybe that's just the rose-tinted glasses of youth in the ZIRP era). This compiler is a toy, it is for all intents and purposes useless except maybe as kind of a piece of art from a bygone time before the Transformer-era of programming. The world we're headed to doesn't really have a place for this kind of thing anymore. We're increasingly headed to a world in which maximally probabilistic hilariously-uninteresting Typescript increasingly runs most of the world as we babysit these token extruders while the economy and markets become increasingly automated by agents. While this makes me kind of sad, it doesn't mean we still can't build this kind of thing [just for fun](https://www.goodreads.com/quotes/10260174-they-are-the-motivational-factors-for-everything-in-your-life-for) and that's what I did here. Maybe the future of functional programming languages is increasingly niche-but-economically-irrelevant and becomes more like a hobby pursuit for the few of us who still love it for the intellectual beauty of the underlying ideas. So I built it anyways, and it runs, and it was a blast to build. And just maybe that's enough, in this brave new world of software.
