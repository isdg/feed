---
title: What's "new" in Miri (and also, there's a Miri paper!)
url: https://www.ralfj.de/blog/2025/12/22/miri.html
published: "2025-12-21T23:00:00Z"
feed: ralfj
guid: https://www.ralfj.de/blog/2025/12/22/miri.html
---

# What's "new" in Miri (and also, there's a Miri paper!)

It is time for another “what is happening in Miri” post. In fact this is *way* overdue, with the [previous update](/blog/2022/07/02/miri.html) being from more than 3 years ago (what even is time?!?), but it is also increasingly hard to find the time to blog, so… here we are. Better late than never. :)

For the uninitiated, [Miri](https://github.com/rust-lang/miri/) is an [Undefined Behavior](https://doc.rust-lang.org/reference/behavior-considered-undefined.html) testing tool for Rust. This means it can find bugs in your unsafe code where you failed to uphold requirements like “all accesses must be aligned” or “mutable references must never alias” or “there must not be any data races”. Miri’s claim to fame is that it is a practical tool that can find *all de-facto Undefined Behavior in deterministic Rust programs*. To my knowledge, no other freely available tool can claim this—for any language.[1](#fn:relwork)

We can only talk about *de-facto UB* because Rust has not yet stabilized its definition of Undefined Behavior. In lieu of that, we carefully check what the compiler does to ensure, to the best of our abilities, that all the UB which a Rust program can encounter *today* is caught by Miri. This means a program that passes Miri should be compiled correctly on *today’s* compiler, but the same program might suffer from UB in the future. Furthermore, if a Rust program is *non-deterministic*, that means it can execute in more than one way, and Miri will only execute it once. You can ask Miri to randomly explore multiple possible executions with `-Zmiri-many-seeds`, but there might always be more executions that Miri did not find yet. This is a fundamental limitation of all testing tools; you generally have to reach for model checking or deductive verification to overcome this.

To learn more about Miri, you can read the [**paper**](https://plf.inf.ethz.ch/research/popl26-miri.html). Yes, there’s a paper! That’s the first bit of news. *“Miri: Practical Undefined Behavior Detection for Rust”* has been accepted to POPL 2026, one of the most prestigious and competitive conferences for fundamental research in Programming Languages.

**Update (2026-02-04):** A recording of the conference talk about this paper is now also [available online](https://www.youtube.com/watch?app=desktop&v=9A8ZeDIStAs).

## Miri progress

The paper aside, what progress has Miri made in the last three years? We have landed over 1500 PRs in that time, so it is impossible to go into all the details, but I will do my best to highlight the general trends and point out anything major.

### Shims

The bread-and-butter of new features added to Miri is to add shims for functions that are implemented outside of Rust and hence cannot be directly executed by Miri. This is mostly about operating system APIs as well as CPU vendor intrinsics. The following list attempts to summarize which shims have been added to Miri since the previous update:

- Greatly expand support for Windows API shims, covering in particular basic file access (by @beepster4096, @CraftSpider).
- Support for various new file descriptor kinds on Unix and specifically Linux, such as `socketpair` (only `SOCK_STREAM`), `pipe`, and `eventfd` (by @DebugSteven, @tiif, @RalfJung, @FrankReh).
- Support for Linux `epoll` (by @tiif with some groundwork and extensions by @DebugSteven, @FrankReh, @RalfJung).
- Broaden the general file API support (by @Pointerbender, @Jefffrey, @tiif, @newpavlov).
- Support for many Intel vendor intrinsics covering SSE2 all the way up to AVX2 (mostly by @eduardosm with some help by @TDecking, @Kixunil). Thanks to @folkertdev, Miri even supports some AVX-512 intrinsics, making it a suitable [testbed](https://trifectatech.org/blog/emulating-avx-512-intrinsics-in-miri/) for code you may not be able to run on real hardware!
- Support for basic functionality on FreeBSD (by @devnexen and @LorrensP-2158466).
- Support for basic functionality on Illumos and Solaris (by @devnexen).
- Support for basic functionality on Android (by @YohDeadfall).
- Improve shims for pthread synchronization operations (by @Mandragorian, @LorrensP-2158466, @RalfJung).
- Extend our support for macOS-specific APIs (by @joboet).
- Support for weak definitions (by @bjorn3).
- Support for miscellaneous small system APIs (by @folkertdev, @Mandragorian, @tgross35, @rayslava, @LorrensP-2158466, @YohDeadfall, @vishruth-thimmaiah, @saethlin, @RalfJung).
- Support for global constructors that execute before `main` (by @ibraheemdev).

### Diagnostics

Our diagnostics have improved dramatically since the last blog post, mostly thanks to @saethlin. For instance, a data race error now points at both of the accesses that caused the race:

```
error: Undefined Behavior: Data race detected between (1) non-atomic read on thread `unnamed-1` and (2) non-atomic write on thread `unnamed-2` at alloc87
  --> tests/fail/data_race/read_write_race.rs:24:13
   |
24 | ...   *c.0 = 64; //~ ERROR: Data race detected between (1) non-atomic read on thread `unnamed-1` and (2) non-atomic write on thread ...
   |       ^^^^^^^^^ (2) just happened here
   |
help: and (1) occurred earlier here
  --> tests/fail/data_race/read_write_race.rs:19:24
   |
19 |             let _val = *c.0;
   |                        ^^^^

```

A use-after-free error shows where the allocation this pointer points to got created and where it got freed:

```
error: Undefined Behavior: memory access failed: alloc194 has been freed, so this pointer is dangling
 --> tests/fail/dangling_pointers/dangling_pointer_deref.rs:9:22
  |
9 |     let x = unsafe { *p }; //~ ERROR: has been freed
  |                      ^^ Undefined Behavior occurred here
  |
help: alloc194 was allocated here:
 --> tests/fail/dangling_pointers/dangling_pointer_deref.rs:6:17
  |
6 |         let b = Box::new(42);
  |                 ^^^^^^^^^^^^
help: alloc194 was deallocated here:
 --> tests/fail/dangling_pointers/dangling_pointer_deref.rs:8:5
  |
8 |     };
  |     ^

```

A Stacked Borrows error shows where the relevant pointer got created, and where it got invalidated:

```
error: Undefined Behavior: attempting a write access using <254> at alloc115[0x0], but that tag does not exist in the borrow stack for this location
 --> tests/fail/stacked_borrows/illegal_write2.rs:8:14
  |
8 |     unsafe { *target2 = 13 }; //~ ERROR: /write access .* tag does not exist in the borrow stack/
  |              ^^^^^^^^^^^^^ this error occurs as part of an access at alloc115[0x0..0x4]
  |
help: <254> was created by a SharedReadWrite retag at offsets [0x0..0x4]
 --> tests/fail/stacked_borrows/illegal_write2.rs:5:19
  |
5 |     let target2 = target as *mut _;
  |                   ^^^^^^
help: <254> was later invalidated at offsets [0x0..0x4] by a Unique retag
 --> tests/fail/stacked_borrows/illegal_write2.rs:6:10
  |
6 |     drop(&mut *target); // reborrow
  |          ^^^^^^^^^^^^

```

@Vanille-N implemented similar tracking for Tree Borrows, so its errors are on par with those emitted by Stacked Borrows. @Zoxc also contributed logic to improve data race errors that involve the aliasing model.

### Optimizations

Miri is still not exactly fast, but some performance work helped significantly speed up Miri’s aliasing checks:

- @saethlin added a garbage collector for pointer tags to Miri, allowing Stacked Borrows to skip a lot of work related to tracking pointers that do not exist anymore.
- @JojoDeveloping added various optimizations to the Tree Borrows checker.

### Improved concurrency support

The data race checker and weak memory support in Miri was originally based on a paper that followed the C++11 concurrency semantics. However, Rust is specified to use the C++20 semantics, which required some adjustments. @cbeuw did the bulk of that work, with help by @SabrinaJewson and @michaliskok. (See [§4 in the paper](https://plf.inf.ethz.ch/research/popl26-miri.html) for more details on this.) As part of writing the paper, I also found and fixed two flaws in the core of the weak memory implementation.

On top of this, @geetanshjuneja adjusted Miri’s scheduler to be fully non-deterministic, making it possible to find issues that would not arise with round-robin scheduling. Furthermore, @pvdrz added support for entirely “virtual” timekeeping to the scheduler, allowing Miri to support a monotone clock in a fully deterministic way. I also made Miri correctly enforce the restrictions for atomic accesses in read-only memory (which is mostly forbidden, with a few exceptions).

Finally, @Patrick-6 has laid the groundwork for integrating GenMC into Miri. GenMC is a weak memory model checker written by @michaliskok, which means that it can enumerate *all* behaviors of a concurrent program (as long as the program has no unbounded loops). By combining it with Miri, we can do full UB checking for all these executions. (When I mentioned model checking in the introduction, that was indeed foreshadowing. :) At the moment, using Miri+GenMC is still highly experimental, slow, and requires a custom build of Miri, but the first steps are taken and I am quite excited about the future potential of this combination!

### Invoking native code from Miri

Did you know that Miri can execute native code invoked from Rust via FFI? This support is very experimental and incomplete, and obviously the native code then runs without any UB checking, but there have been significant improvements since the previous update:

- @Strophox has implemented support for sharing Rust-allocated memory with the native code.
- @nia-e has added some truly cursed magic to let Miri trace quite precisely which memory the native code accesses, and use that to improve Miri’s UB checking. I hear she has plans for making this even more powerful in the future. :)

### Miscellaneous

Finally, we had some contributions that are large enough to be worth mentioning, but did not fit any of the categories above:

- The memory leak detector can now take into account main-thread thread-local storage (by @max-heller).
- Our representation of file descriptions is a lot more flexible and extensible (by @Luv-Ray, @oli-obk, @RalfJung).
- Miri now makes the precision of some floating-point operations non-deterministic to catch code incorrectly relying on precise or deterministic answers (by @LorrensP-2158466).
- Miri non-deterministically causes `read` and `write` operations to only process a part of the buffer to catch programs that incorrectly rely on such operations reliably completing immediately (by @RalfJung).
- Tree Borrows now by default tracks `UnsafeCell` as precisely as Stacked Borrows, catching UB when other bytes behind the same reference are incorrectly mutated (by @yoctocell, @JojoDeveloping).
- Tree Borrows now supports wildcard provenance, so Miri can check programs that use integer-to-pointer casts and still catch some of the bugs involving those pointers (by @royAmmerschuber).
- Miri can detect UB related to in-place (“move”) function arguments (by @RalfJung).
- Miri supports precise profiling, tracking where all the execution time is spent (by @Stypox).
- Miri no longer needs xargo, cutting down the setup effort (by @RalfJung).

On top of this, there has been a lot of bug-fixing as well as continuous refactoring and cleanup to keep the code maintainable. Thank you to everyone who contributed. If your name should be on that list, then I am sorry for forgetting you.

## How you can help

If you want to help improve Miri, that’s awesome! The [issue tracker](https://github.com/rust-lang/miri/issues) is a good place to start; the list of issues is short enough that you can just browse through it rather quickly to see if anything piques your interest. The ones that are particularly suited for getting started are marked with a green label. Another good starting point is to try to implement the missing bit of functionality that keeps your test suite from working. That said, you should have gathered some Rust experience in a simpler project before tackling Miri; Miri is not a good codebase for your first steps in Rust. However, if you already know Rust, Miri can be a fun and rewarding next challenge! If you need any mentoring, just [get in touch](https://rust-lang.zulipchat.com/#narrow/stream/269128-miri). :)

We are currently looking for someone who wants to maintain Miri’s support for wasm targets. The wasm API is quite different from other OS APIs, so maintaining those shims without the help of someone who actually knows and understands the wasm ecosystem turned out to not be sustainable. Furthermore, we are also looking for someone with Android experience to serve as Android target maintainer for Miri. This mostly means fixing Android-specific issues that come up due to changes in the standard library – which should be rare, but when it happens it is really useful to have someone to ping. Android is also fairly close to passing our entire test suite, so you could make it your challenge to fix the remaining bits and pieces.

That’s all for now! I am immensely proud of what Miri has become, and deeply grateful to everyone who helped us get here. Thanks to Miri and other tools based on Miri’s understanding of UB, avoiding Undefined Behavior in unsafe code becomes practically possible even at scale, while at the same time giving us a chance to specify UB in a way that satisfies the needs of real-world unsafe code out there. This has worked out way better than I could have ever imagined, and it would not have been possible without the entire amazing Rust community being on-board – I am looking forward to seeing what’s next. :D

1. Please let me know if there is such a tool and I just missed it! The paper discusses why sanitizers and valgrind, while being immensely useful tools, still miss some UB. I just know of one commercial tool that can make similar claims to Miri, the “TrustInSoft Analyzer”. It requires a license, so I can’t say how good its coverage of the C standard is; in particular, it would be interesting to compare what GCC and clang think is UB with what the tool thinks. In Miri, we spend a lot of time discussing with compiler folks to ensure we all have a shared understanding of what is and is not UB. In principle, this should not be needed in C since it has a standard; in practice, the standard can [diverge pretty far](https://dl.acm.org/doi/10.1145/2908080.2908081) from what programmers expect and from what compilers implement. [↩](#fnref:relwork)
