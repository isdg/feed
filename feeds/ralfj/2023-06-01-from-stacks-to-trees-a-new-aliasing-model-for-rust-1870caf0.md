---
title: 'From Stacks to Trees: A new aliasing model for Rust'
url: https://www.ralfj.de/blog/2023/06/02/tree-borrows.html
published: "2023-06-01T22:00:00Z"
feed: ralfj
guid: https://www.ralfj.de/blog/2023/06/02/tree-borrows.html
---

# From Stacks to Trees: A new aliasing model for Rust

Since last fall, [Neven](https://perso.crans.org/vanille/) has been doing an internship to develop a new aliasing model for Rust: Tree Borrows. Hang on a second, I hear you say – doesn’t Rust already have an aliasing model? Isn’t there this “Stacked Borrows” that Ralf keeps talking about? Indeed there is, but Stacked Borrows is just one proposal for a possible aliasing model – and it [has](https://github.com/rust-lang/unsafe-code-guidelines/issues/133) [its](https://github.com/rust-lang/unsafe-code-guidelines/issues/134) [fair](https://github.com/rust-lang/unsafe-code-guidelines/issues/256) [share](https://github.com/rust-lang/unsafe-code-guidelines/issues/274) [of](https://github.com/rust-lang/unsafe-code-guidelines/issues/276) [problems](https://github.com/rust-lang/unsafe-code-guidelines/issues/303). The purpose of Tree Borrows is to take the lessons learned from Stacked Borrows to build a new model with fewer issues, and to take some different design decisions such that we get an idea of some of the trade-offs and fine-tuning we might do with these models before deciding on the official model for Rust.

Neven has written a detailed introduction to Tree Borrows [on his blog](https://perso.crans.org/vanille/treebor/), which you should go read first. He presented this talk at a recent RFMIG meeting, so you can also [watch his talk here](https://www.youtube.com/watch?v=zQ76zLXesxA). In this post, I will focus on the differences to Stacked Borrows. I assume you already know Stacked Borrows and want to understand what changes with Tree Borrows and why.

As a short-hand, I will sometimes write SB for Stacked Borrows and TB for Tree Borrows.

## Two-phase borrows

The main novelty in Tree Borrows is that it comes with proper support for two-phase borrows. Two-phase borrows are a mechanism introduced with NLL which allows code like the following to be accepted:

```
fn two_phase(mut x: Vec<usize>) {
    x.push(x.len());
}

```

The reason this code is tricky is that it desugars to something like this:

```
fn two_phase(mut x: Vec<usize>) {
    let arg0 = &mut x;
    let arg1 = Vec::len(&x);
    Vec::push(arg0, arg1);
}

```

This code clearly violates the regular borrow checking rules since `x` is mutably borrowed to `arg0` when we call `x.len()`! And yet, the compiler will accept this code. The way this works is that the `&mut x` stored in `arg0` is split into two phases: in the *reservation* phase, `x` can still be read via other references. Only when we actually need to write to `arg0` (or call a function that might write to it) will the reference be “activated”, and it is from that point onwards (until the end of the lifetime of the borrow) that no access via other references is allowed. For more details, see [the RFC](https://github.com/rust-lang/rfcs/blob/master/text/2025-nested-method-calls.md) and [the rustc-dev-guide chapter on two-phase borrows](https://rustc-dev-guide.rust-lang.org/borrow_check/two_phase_borrows.html). The only point relevant for this blog post is that when borrowing happens implicitly for a method call (such as `x.push(...)`), Rust will treat this as a two-phase borrow. When you write `&mut` in your code, it is treated as a regular mutable reference without a “reservation” phase.

For the aliasing model, two-phase borrows are a big problem: by the time `x.len()` gets executed, `arg0` already exists, and as a mutable reference it really isn’t supposed to allow reads through other pointers. Therefore Stacked Borrows just [gives up](https://github.com/rust-lang/unsafe-code-guidelines/issues/85) here and basically treats two-phase borrows like raw pointers. That is of course unsatisfying, so for Tree Borrows we are adding proper support for two-phase borrows. What’s more, we are treating *all* mutable references as two-phase borrows: this is more permissive than what the borrow checker accepts, but lets us treat mutable references entirely uniformly. (This is a point we might want to tweak, but as we will see soon this decision actually has some major unexpected benefits.)

This is why we need a tree in the first place: `arg0` and the reference passed to `Vec::len` are both children of `x`. A stack is no longer sufficient to represent the parent-child relationships here. Once the use of a tree is established, modeling of two-phase borrows is fairly intuitive: they start out in a `Reserved` state which tolerates reads from other, unrelated pointers. Only when the reference (or one of its children) is written to for the first time, its state transitions to `Active` and now reads from other, unrelated pointers are not accepted any more. (See Neven’s post for more details. In particular note that there is one unpleasant surprise lurking here: if there are `UnsafeCell` involved, then a reserved mutable reference actually has to tolerate *mutation* via unrelated pointers! In other words, the aliasing rules of `&mut T` are now affected by the presence of `UnsafeCell`. I don’t think people realized this when two-phase borrows were introduced, but it also seems hard to avoid so even with hindsight, it is not clear what the alternative would have been.)

## Delayed uniqueness of mutable references

One of the most common source of Stacked Borrows issues is its [very eager enforcement of uniqueness of mutable references](https://github.com/rust-lang/unsafe-code-guidelines/issues/133). For example, the following code is illegal under Stacked Borrows:

```
let mut a = [0, 1];
let from = a.as_ptr();
let to = a.as_mut_ptr().add(1); // `from` gets invalidated here
std::ptr::copy_nonoverlapping(from, to, 1);

```

The reason it is illegal is that `as_mut_ptr` takes `&mut self`, which asserts unique access to the entire array, therefore invalidating the previously created `from` pointer. In Tree Borrows, however, that `&mut self` is a two-phase borrow! `as_mut_ptr` does not actually perform any writes, so the reference remains reserved and never gets activated. That means the `from` pointer remains valid and the entire program is well-defined. The call to `as_mut_ptr` is treated like a read of `*self`, but `from` (and the shared reference it is derived from) are perfectly fine with reads via unrelated pointers.

It happens to be the case that swapping the `from` and `to` lines actually makes this code work in Stacked Borrows. However, this is not for a good reason: this is a consequence of the rather not-stack-like rule in SB which says that on a read, we merely *disable all `Unique`* above the tag used for the access, but we keep raw pointers derived from those `Unique` pointers enabled. Basically, raw pointers can live longer than the mutable references they are derived from, which is highly non-intuitive and potentially problematic for program analyses. With TB, the swapped program is still fine, but for a different reason: when `to` gets created first, it remains a reserved two-phase borrow. This means that creating a shared reference and deriving `from` from it (which acts like a read on `self`) is fine; reserved two-phase borrows tolerate reads via unrelated pointers. Only when `to` is written to does it (or rather the `&mut self` it was created from) become an active mutable reference that requires uniqueness, but that is after `as_ptr` returns so there is no conflicting `&self` reference.

It turns out that consistently using two-phase borrows lets us entirely eliminate this hacky SB rule and also fix one of the most common sources of UB under SB. I didn’t expect this at all, so this is a happy little accident. :)

However, note that the following program is fine under SB but invalid under TB:

```
let mut a = [0, 1];
let to = a.as_mut_ptr().add(1);
to.write(0);
let from = a.as_ptr();
std::ptr::copy_nonoverlapping(from, to, 1);

```

Here, the write to `to` activates the two-phase borrow, so uniqueness is enforced. That means the `&self` created for `as_ptr` (which is considered reading all of `self`) is incompatible with `to`, and so `to` is invalidated (well, it is made read-only) when `from` gets created. So far, we do not have evidence that this pattern is common in the wild. The way to avoid issues like the code above is to *set up all your raw pointers before you start doing anything*. Under TB, calling reference-receiving methods like `as_ptr` and `as_mut_ptr` and using the raw pointers they return on disjoint locations is fine even if these references overlap, but you must call all those methods before the first write to a raw pointer. Once the first write happens, creating more references can cause aliasing violations.

## No strict confinement of the accessible memory range

The other major source of trouble with Stacked Borrows is [restricting raw pointers to the type and mutability they are initially created with](https://github.com/rust-lang/unsafe-code-guidelines/issues/134). Under SB, when a reference is cast to `*mut T`, the resulting raw pointer is confined to access only the memory covered by `T`. This regularly trips people up when they take a raw pointer to one element of an array (or one field of a struct) and then use pointer arithmetic to access neighboring elements. Moreover, when a reference is cast to `*const T`, it is actually read-only, even if the reference was mutable! Many people expect `*const` vs `*mut` not to matter for aliasing, so this is a regular source of confusion.

Under TB, we resolve this by no longer doing any retagging for reference-to-raw-pointer casts. A raw pointer simply uses the same tag as the parent reference it is derived from, thereby inheriting its mutability and the range of addresses it can access. Moreover, references are not strictly confined to the memory range described by their type: when an `&mut T` (or `&T`) gets created from a parent pointer, we initially record the new reference to be allowed to access the memory range describe by `T` (and we consider this a read access for that memory range). However, we also perform *lazy initialization*: when a memory location outside this initial range is accessed, we check if the parent pointer would have had access to that location, and if so then we also give the child the same access. This is repeated recursively until we find a parent that has sufficient access, or we reach the root of the tree.

This means TB is compatible with [`container_of`-style pointer arithmetic](https://github.com/rust-lang/unsafe-code-guidelines/issues/243) and [`extern` types](https://github.com/rust-lang/unsafe-code-guidelines/issues/276), overcoming two more SB limitations.

This also means that the following code becomes legal under TB:

```
let mut x = 0;
let ptr = std::ptr::addr_of_mut!(x);
x = 1;
ptr.read();

```

Under SB, `ptr` and direct access to the local `x` used two different tags, so writing to the local invalidated all pointers to it. Under TB, this is no longer the case; a raw pointer directly created to the local is allowed to alias arbitrarily with direct accesses to the local.

Arguably the TB behavior is more intuitive, but it means we can no longer use writes to local variables as a signal that all possible aliases have been invalidated. However, note that TB only allows this if there is an `addr_of_mut` (or `addr_of`) immediately in the body of a function! If a reference `&mut x` is created, and then some other function derives a raw pointer from that, those raw pointers *do* get invalidated on the next write to `x`. So to me this is a perfect compromise: code that uses raw pointers has a lower risk of UB, but code that does not use raw pointers (which is easy to see syntactically) can be optimized as much as with SB.

Note that this entire approach in TB relies on TB *not* needing the stack-violating hack mentioned in the previous section. If raw pointers in SB just inherited their parent tag, then they would get invalidated together with the unique pointer they are derived from, disallowing all the code that this hack was specifically added to support. This means that backporting these improvements to SB is unlikely to be possible.

## `UnsafeCell`

The handling of `UnsafeCell` also changed quite a bit with TB. First of all, another [major issue](https://github.com/rust-lang/unsafe-code-guidelines/issues/303) with SB was fixed: turning an `&i32` into an `&Cell<i32>` *and then never writing to it* is finally allowed. This falls out of how TB handles the aliasing allowed with `UnsafeCell`: they are treated like casts to raw pointers, so reborrowing an `&Cell<i32>` just inherits the tag (and therefore the permissions) of the parent pointer.

More controversially, TB also changes how precisely things become read-only when an `&T` involves `UnsafeCell` somewhere inside `T`. In particular, for `&(i32, Cell<i32>)`, TB allows mutating *both* fields, including the first field which is a regular `i32`, since it just treats the entire reference as “this allows aliasing”.[1](#fn:1) In contrast, SB actually figured out that the first 4 bytes are read-only and only the last 4 bytes allow mutation via aliased pointers.

The reason for this design decision is that the general philosophy with TB was to err on the side of allowing more code, having less UB (which is the opposite direction than what I used with SB). This is a deliberate choice to uncover as much of the design space as we can with these two models. Of course we wanted to make sure that TB still allows all the desired optimizations, and still has enough UB to justify the LLVM IR that rustc generates – those were our “lower bounds” for the minimum amount of UB we need. And it turns out that under these constraints, we can support `UnsafeCell` with a fairly simple approach: for the aliasing rules of `&T`, there are only 2 cases. Either there is no `UnsafeCell` anywhere, then this reference is read-only, or else the reference allows aliasing. As someone who thinks a lot about proving theorems about the full Rust semantics including its aliasing model, this approach seemed pleasingly simple. :)

I expected this decision to be somewhat controversial, but the amount of pushback we received has still been surprising. The good news is that this is far from set in stone: we can [change TB to treat `UnsafeCell` more like SB did](https://github.com/rust-lang/unsafe-code-guidelines/issues/403). Unlike the previously described differences, this one is entirely independent of our other design choices. While I prefer the TB approach, the way things currently stand, I do expect that we will end up with SB-like `UnsafeCell` treatment eventually.

## What about optimizations?

I have written a lot about how TB differs from SB in terms of which coding patterns are UB. But what about the other side of the coin, the optimizations? Clearly, since SB has more UB, we have to expect TB to allow fewer optimizations. And indeed there is a major class of optimizations that TB loses: speculative writes, i.e. inserting writes in code paths that would not previously have written to this location. This is a powerful optimization and I was quite happy that SB could pull it off, but it also comes at a major cost: mutable references have to be “immediately unique”. Given how common of a problem “overeager uniqueness” is, my current inclination is that we most likely would rather make all that code legal than allow speculative writes. We still have extremely powerful optimization principles around reads, and when the code *does* perform a write that gives rise to even more optimizations, so my feeling is that insisting on speculative writes is just pushing things too far.

On another front, TB actually allows a set of crucial optimizations that SB ruled out by accident: reordering of reads! The issue with SB is that if we start with “read mutable reference, then read shared reference”, and then reorder to “read shared reference, then read mutable reference”, then in the new program, reading the shared reference might invalidate the mutable reference – so the reordering might have introduced UB! This optimization is possible without having any special aliasing model, so SB not allowing it is a rather embarrassing problem. If it weren’t for the stack-violating hack that already came up several times above, I think there would be a fairly easy way of fixing this problem in SB, but alas, that hack is load-bearing and too much existing code is UB if we remove it. Meanwhile, TB does not need any such hack, so we can do the Right Thing (TM): when doing a read, unrelated mutable references are not entirely disabled, they are just made read-only. This means that “read shared reference, then read mutable reference” is equivalent to “read mutable reference, then read shared reference” and the optimization is saved. (A consequence of this is that retags can also be reordered with each other, since they also act as reads. Hence the order in which you set up various pointers cannot matter, until you do the first write access with one of them.)

## Future possibility: `Unique`

Tree Borrows paves the way for an extension that we have not yet implemented, but that I am quite excited to explore: giving meaning to `Unique`. `Unique` is a private type in the Rust standard library that was originally meant to express `noalias` requirements. However, it was never actually wired up to emit that attribute on the LLVM level. `Unique` is mainly used in two places in the standard library: `Box` and `Vec`. SB (and TB) treat `Box` special (matching rustc itself), but not `Unique`, so `Vec` does not come with any aliasing requirements. And indeed the SB approach totally does not work for `Vec`, since we don’t actually know how much memory to make unique here. However, with TB we have lazy initialization, so we don’t need to commit to a memory range upfront – we can make it unique “when accessed”. This means we can explore giving meaning to the `Unique` in `Vec`.

Now, this might not actually work. People actually do blatantly-aliasing things with `Vec`, e.g. to implement arenas. On the other hand, `Vec`’s uniqueness would only come in when it is moved or passed *by value*, and only for the memory ranges that are actually being accessed. So it is quite possible that this is compatible with arenas. I think the best way to find out is to implement `Unique` semantics behind a flag and experiment. If that works out, we might even be able to remove all special handling of `Box` and rely on the fact that `Box` is defined as a newtype over `Unique`. This would slightly reduce the optimization potential ( `Box<T>` is known to point to a memory range at least the size of `T`, whereas `Unique` has no such information), but making `Box` less magic is a long-standing quest so this might be an acceptable trade-off.

I should note that there are many people who think neither `Box` nor `Vec` should have any aliasing requirements. I think it’s worth first exploring whether we can have aliasing requirements which are sufficiently light-weight that they are compatible with common coding patterns, but even if we end up saying `Box` and `Vec` behave like raw pointers, it can still be useful to have `Unique` in our toolbox and expose it for unsafe code authors to eke out the last bits of performance.

## Conclusion

These are the major differences between Stacked Borrows and Tree Borrows. As you can see, almost all of them are cases where TB allows more code than SB, and indeed TB fixes what I consider to be SB’s two biggest problems: overeager uniqueness for mutable references, and confining references and raw pointers to the size of the type they are created with. These are great news for unsafe code authors!

What TB *doesn’t* change is the presence of “protectors” to enforce that certain references remain valid for the duration of an entire function call (whether they are used again or not); protectors are absolutely required to justify the LLVM `noalias` annotations we would like to emit and they also do enable some stronger optimizations than what would otherwise be possible. I do expect protectors to be the main remaining source of unexpected UB from Tree Borrows, and I don’t think there is a lot of wiggle-room that we have here, so this might just be a case where we have to tell programmers to adjust their code, and invest in documentation material to make this subtle issue as widely known as possible.

Neven has implemented Tree Borrows in Miri, so you can play around with it and check your own code by setting `MIRIFLAGS=-Zmiri-tree-borrows`. If you run into any surprises or concerns, please let us know! The [t-opsem Zulip](https://rust-lang.zulipchat.com/#narrow/stream/136281-t-opsem) and the [UCG issue tracker](https://github.com/rust-lang/unsafe-code-guidelines/) are good places for such questions.

That’s all I got, thanks for reading – and a shout out to Neven for doing all the actual work here (and for giving feedback on this blog post), supervising this project has been a lot of fun! Remember to read [his write up](https://perso.crans.org/vanille/treebor/) and [watch his talk](https://www.youtube.com/watch?v=zQ76zLXesxA).

1. This does not mean that we bless such mutation! It just means that the compiler cannot use immutability of the first field for its optimizations. Basically, immutability of that field becomes a [safety invariant instead of a validity invariant](/blog/2018/08/22/two-kinds-of-invariants.html): when you call foreign code, you can still rely on it not mutating that field, but within the privacy of your own code you are allowed to mutate it. See [my response here](https://www.reddit.com/r/rust/comments/13y8a9b/comment/jmlvgun/) for some more background. [↩](#fnref:1)
