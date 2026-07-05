---
title: How to use storytelling to fit inline assembly into Rust
url: https://www.ralfj.de/blog/2026/03/13/inline-asm.html
published: "2026-03-12T23:00:00Z"
feed: ralfj
guid: https://www.ralfj.de/blog/2026/03/13/inline-asm.html
---

# How to use storytelling to fit inline assembly into Rust

The Rust Abstract Machine is full of [wonderful oddities](/blog/2020/12/14/provenance.html) that do not exist on the [actual hardware](/blog/2019/07/14/uninit.html). Inevitably, every time this is discussed, someone asks: “But, what if I use inline assembly? What happens with provenance and uninitialized memory and Tree Borrows and all these other fun things you made up that don’t actually exist?” This is a great question, but answering it properly requires some effort. In this post, I will lay down my current thinking on how inline assembly fits into the Rust Abstract Machine by giving a *general principle* that explains how anything we decide about the semantics of pure Rust impacts what inline assembly may or may not do.

Note that everything I discuss here applies to FFI calls just as much as it applies to inline assembly. Those mechanisms are fundamentally very similar: they allow Rust code to invoke code not written in Rust.[1](#fn:xlang) I will not keep repeating “inline assembly or FFI” throughout the post, but every time I refer to inline assembly this is meant to also include FFI.

To get started, let me explain why there are things that even inline assembly is fundamentally not allowed to do.

## Why can’t inline assembly do whatever it wants?

People like to think of inline assembly as freeing them from all the complicated requirements of the Abstract Machine. Unfortunately, that’s a pipe dream. Here is an example to demonstrate this:

```
use std::arch::asm;

#[inline(never)]
fn innocent(x: &i32) { unsafe {
    // Store 0 at the address given by x.
    asm!(
        "mov dword ptr [{x}], 0",
        x = in(reg) x,
    );
} }

fn main() {
    let x = 1;
    innocent(&x);
    assert!(x == 1);
}

```

When the compiler analyzes `main`, it realizes that only a shared reference is being passed to `innocent`. This means that whatever `innocent` does, it cannot change the value stored at `*x`. Therefore, the assertion can be optimized away.

However, `innocent` actually does write to `*x`! Therefore, the optimization changed the behavior of the program. And indeed, this is [exactly what happens](https://rust.godbolt.org/z/GG7YPsEcs) with current versions of rustc: without optimizations, the assertion fails, but with optimizations, it passes. Therefore, either the optimization was wrong, or the program had Undefined Behavior. And since this is an optimization that we really want to be able to perform, we can only pick the second option.[2](#fn:nondet)

However, where does the UB come from? If the entire program was written in Rust, the answer would be “the aliasing model”. Both Stacked Borrows and Tree Borrows, and any other aliasing model worth considering for Rust, will make it UB to write through pointers derived from a shared reference. However, this time, parts of the program are not written in Rust, so things are not that simple. How can we say that the inline asm block violated Tree Borrows, when it is written in a language that does not have anything even remotely comparable to Tree Borrows? That’s what the rest of this post is about.

I hope the example clearly demonstrates that we *cannot* get away with having inline assembly just ignore Abstract Machine concepts such as Tree Borrows. The inline asm block causes UB, we just have to figure out how and why—and more importantly, we have to figure out how people can ensure that their inline asm blocks do *not* cause UB.

## When is inline assembly compatible with optimizations?

It may seem like we now have to define a version of Tree Borrows that works with assembly code. That would be an impossible task (Tree Borrows relies on pointer provenance, which does not exist in assembly).[3](#fn:cheri) Lucky enough, this is also not necessary.

Instead, we can piggy-back on the already existing definition of Tree Borrows and the rest of the Abstract Machine. We do this by requiring the programmer to *tell a story* about what the inline assembly block does in Rust terms.[4](#fn:alice) (If this sounds strange, please bear with me. I will explain why this makes sense.) Specifically, for every inline assembly block, there has to be a corresponding piece of Rust code that *does the same thing* ***as far as the state observable by pure Rust code is concerned***. When reasoning about the behavior of the overall program, the inline assembly block then gets replaced by that “story” code. You don’t have to actually write this code; what’s important is that the code exists and tells a coherent story with what the surrounding Rust code does.

For our example above, this immediately explains what went wrong: the story code for the inline assembly block would have to be something like `(x as *const i32 as *mut i32).write(0)`, and if we insert that code in place of the inline assembly block, we can immediately see (and Miri could confirm) that the program has UB. An inline assembly block can have many possible stories, and it is enough to find *one* story that makes everything work, but in this case, that is not possible.

So, in slightly more detail, here is what I consider to be the rules for inline assembly:

1. For every inline assembly block, pick a “story”: a piece of Rust code that serves as stand-in for what this inline assembly block does *to the Abstract Machine state*. This story code only has access to data that is made available to the inline assembly block (explicit operands and global variables). When reasoning about soundness and correctness of the program on the Abstract Machine level, we pretend that the story code gets executed instead of the assembly code.
2. This piece of code has to satisfy all the requirements that are imposed on the asm block by attributes such as `readonly` or `nomem` and honor operand constraints such as not mutating `in` operands.
3. The actual assembly code has to *refine* the story code, i.e., whatever the assembly code does to state which the Abstract Machine can observe (in particular, operands and global variables) has to be something that the story code could also have done.

I should add the disclaimer that I do not have a formal theory that proves correctness of this approach. However, I am reasonably confident, because this approach fits in very well with how we prove the correctness for optimizations such as the one in our example above: At the heart of the correctness argument is a proof that *all* Rust code satisfies some universal properties. For instance, we can formalize and prove the claim that any Rust function which takes a shared reference without interior mutability as argument cannot write to that argument. This isn’t the only such property; in fact the set of such properties isn’t fully known: we might discover a new property upheld by all Rust code tomorrow. What’s crucial is that any property of the form “for all Rust programs, …” must also hold for the story code, since that is just regular Rust code! Finally, because the actual assembly code refines the story code, we know that for the purpose of reasoning about the program, we can pretend that actually the story code gets executed and then, at the end of compilation, replace the story code by the desired assembly code without changing program behavior.

So, that is why story code works. But, doesn’t this make inline assembly entirely useless? After all, the entire point of inline assembly is to do things I couldn’t already do in pure Rust!

## Inline assembly stories by example

To convince you that the storytelling approach is feasible, let us consider a few representative examples for inline assembly usage and what the corresponding story might look like.

#### Pure instructions

The easiest case is code that wants to access a new hardware operation that is not exposed in the language. For instance, the inline assembly block might consist of a single instruction that returns the number of bits set to 1 in a register. Here, storytelling is trivial: we can just write some Rust code that does a bit of bit manipulation by hand to count the number of bits set to 1.

#### Page table manipulation

That was easy, so let us crank up the difficulty and consider an OS kernel that manipulates page tables. Rust has no notion of page tables. What could the “story” possibly look like here?

The answer is that Rust has something that is very similar to putting a new page into the page table—it is called `alloc`. It also has something very similar to removing a page ( `dealloc`), and to moving a page to a different location in the address space ( `realloc`). So, the story that an OS kernel would tell the compiler is that manipulating page tables is really just a funny kind of allocator.

Slightly more concretely, “allocating” a page in a way that is compatible with the storytelling approach could look like this:

- First, some Rust code performs the actual page table manipulation using volatile loads and stores.[5](#fn:volatile)
- Then an asm block executes whatever barrier is needed on the current system to ensure the updated page table has taken effect.
- Next, the address of the page is cast to a pointer (using [`with_exposed_provenance`](https://doc.rust-lang.org/std/ptr/fn.with_exposed_provenance.html)).
- Finally, Rust code may use that pointer to access the new page.

The story of this asm block is that it performs memory allocation at the given address, which we know to be unallocated.[6](#fn:alloc-control) This creates a fresh provenance that represents the new allocation. This allocation is then immediately [exposed](https://doc.rust-lang.org/std/primitive.pointer.html#method.expose_provenance) by the story code.

Even for architectures where no barrier is needed after a page table change, the asm block is still crucial: it prevents the compiler from reordering accesses to the new pages up across the page table manipulation! Using the usual rules for Rust programs, there is no way the compiler could figure out that there is any sort of dependency here. The asm block therefore serves as a compiler fence: as far as the compiler is concerned, this block might actually invoke the story code we made up, and therefore the new pointer and operations based on it cannot be moved to before the asm block.

This is why people sometimes think of asm blocks as compiler fences: an asm block stands in for some arbitrary “story code” the compiler doesn’t know, so the compiler has to treat this code *as if* some arbitrary code was executing here, which prevents most reorderings. But the emphasis here is on *most*: if the compiler has extra aliasing information, such as from an `&mut` type, that lets the compiler reason about and reorder memory accesses even across unknown function calls and, therefore, inline asm blocks. It is therefore incorrect to say that an asm block is a barrier preventing all reordering. Thinking in terms of compiler barriers can provide useful intuition, but a rigorous correctness argument needs to go into more detail.

There is another caveat in this story: with page table manipulation, one cannot just create new allocations, one can also grow and shrink existing allocations. In fact, the same is possible from userspace with `mmap`. It turns out that *growing* allocations is harmless, so this has been [officially blessed](https://github.com/llvm/llvm-project/pull/141338) in LLVM and we should find a way to also expose this on the Rust side. However, *shrinking* allocations is problematic—there are [simple optimizations](https://github.com/llvm/llvm-project/pull/141338) that LLVM might reasonably perform that would break code which shrinks allocations! So, further work is required to ensure that Rust code (as well as C and C++ code) can use `munmap` without risking miscompilations. This is why it is so important to take a principled approach to language semantics and correctness: it would otherwise be way too easy to miss potential problems like this.

#### Page table manipulation II: duplicating pages

Next, let us consider another case of page table shenanigans: mapping a single page of physical memory into multiple locations in virtual memory. That means the page is “mirrored” in multiple places, and mutating any one mirror changes all of them. First of all, note that in general, this is plain unsound. LLVM will freely assume that `ptr` and `ptr.wrapping_offset(4096)` do not alias, so mapping the same memory into multiple places and freely accessing all of them can lead to subtle miscompilations. However, there is a restricted form of this where we can use inline assembly to come up with a “story” that fits into the Abstract Machine, and is therefore sound.

The key limitation is that the program only gets to use one of the “mirrored” version of this memory at a time. Changing which mirror is “active” requires an explicit barrier and returns a new pointer that has to be used for future accesses. This barrier can be an empty inline assembly block that just returns the pointer unchanged, but the story we attach to it is all but empty: we will say that this behaves like a `realloc`, logically moving the allocation from one mirror to another. In other words, as far as the Rust Abstract Machine is concerned, only one of the mirrored versions of memory actually “exists”, and switching to a different one amounts to freeing the old allocation and creating a new one. Crucially, as usual with `realloc`, after each such switch all the old pointers to that memory become invalid and the new pointer returned by the switch is the only way to access that memory.[7](#fn:intptr) These inline asm blocks will also prevent LLVM from reordering accesses to different “mirrors” around each other, thus avoiding the aforementioned miscompilations. In other words, changing our code in a way that lets us tell a proper story also introduced enough structure to prevent the optimizer from doing things it shouldn’t do.

This may sound a bit contrived, but such a “purely logical” `realloc` actually comes up in more than one situation; there even is [an open RFC](https://github.com/rust-lang/rfcs/pull/3700) proposing to add it to the language proper.

#### Non-temporal stores

The previous example already showed that some hardware features are too intrusive to be freely available inside a high-level language such as Rust. Non-temporal stores are another example of this. Specifically, I am referring to the “streaming” store operations on x86 ( `_mm_stream_ps` and friends). The main point of these operations is to avoid cluttering the cache with data that is unlikely to be read again soon, but they also have the unfortunate side effect of breaking the usual “total store order” memory model of x86. This is bad news because the compilation of the rest of the program relies on that memory model.

To explain the problem, let us consider what the “story” for a non-temporal store might be. The obvious choice is to make it just a regular write access—caching is not modeled in the Abstract Machine, after all. Unfortunately, this does not work. Consider the case where the streaming store is followed by an atomic release write. Due to the total store order model of x86, this compiles to a regular write instruction without any extra fences. However, streaming stores actually *do* require a fence ( `_mm_sfence`) for proper synchronization. Therefore, one can write a Rust program that seems to be data-race-free (according to the story) but actually has a data race. In other words, rule 3 (the inline asm block must refine the story code) is violated.

The principled fix for this is to extend the C++ memory model (which is shared by Rust) with a notion of non-temporal stores so that one can reason about how they interact with everything else that can go on in a concurrent program. This is [possible](https://people.mpi-sws.org/~viktor/papers/oopsla2024-inline.pdf), but it requires re-proving compiler correctness results, and at least the approach taken in that paper is architecture-specific which does not scale to the large number of architectures Rust supports. However, there is a simpler alternative: we can try coming up with a more complicated story such that rule 3 is not violated. This is exactly what a bunch of folks did when the issue around non-temporal stores was discovered. The story says that doing a non-temporal store corresponds to *spawning a thread* that will asynchronously perform the actual store, and `_mm_sfence` corresponds to waiting for all those threads to finish. This explains why release-acquire synchronization fails: synchronization picks up all writes performed by the releasing thread, but the streaming store conceptually happened on a different thread! The new story code was the basis for the updated [documentation](https://doc.rust-lang.org/nightly/core/arch/x86/fn._mm_sfence.html) for streaming stores on x86, and the code itself can even be found in a [comment in the code](https://github.com/rust-lang/rust/blob/cb3046e5f2f0736366c0fea4977a8df579d96311/library/stdarch/crates/core_arch/src/x86/sse.rs#L1456-L1481).

There is one caveat: The story we picked implies that it is UB for the thread that performed the streaming store to do a load from that memory before `_mm_sfence`, even though this operation would be well-defined on the underlying hardware. This is the price we pay in exchange for having a principled argument for why code using streaming stores will not be miscompiled. It is not a high price: streaming stores are used for data that is unlikely to be read again soon, that is their entire point. None of the examples of streaming stores we found in the wild had a problem with this limitation.[8](#fn:forgotten-fence)

#### Stack painting

Another possible use for inline assembly is measuring the stack consumption of a program using stack painting. This was brought up as a question in the t-opsem Zulip, and I am including it here because it is a nice demonstration of how much freedom the storytelling approach provides, and which limitations it has.

Roughly speaking, stack painting means that before the program starts, the memory region that will later become the stack is filled with a fixed bitpattern. Later, we can then measure the maximum stack usage of the program by checking where the bit pattern is still intact and where it has been overwritten. This can be done with inline assembly code that simply directly reads from the stack.

The first reflex might be to say that this is obviously UB: that stack memory might be subject to noalias constraints (due to a mutable reference pointing to the stack); you can’t just read from memory that you don’t have permission to read. However, that presupposes that the story for this asm block involves reading memory. An alternative story is to say that the asm block just returns some arbitrary, non-deterministically chosen value. The upside of this story is that, as long as the read doesn’t trap, the story is always correct according to our rules: whatever the assembly code actually does, it surely refines returning an arbitrary value. However, the downside of this story is that when reasoning about our code, we cannot make *any* assumptions about the value we read! Correctness of our program is defined under the storytelling semantics, i.e., the program has to be correct no matter which values are returned by the inline asm. That may sound like a problem, but for this use case, it is actually entirely fine: stack painting anyway provides just an estimate of the real stack usage. The compiler makes no *guarantees* that the measurement produced this way is remotely accurate, but experiments show that this works well in practice. Incorrect measurements do not lead to soundness or correctness issues, so providing accurate answers is “just” a quality-of-life concern.

#### Floating-point status and control register

The final example I want to consider are floating-point status and control registers. This is an example where the storytelling approach mostly serves to explain why using these registers is not possible or not useful.

Programmers sometimes want to read the status register to check if a floating-point exception has been raised, and to write the control register to adjust the rounding mode or other aspects of floating-point computations. However, actually supporting such control register mutations is a disaster for optimizations: the control register is global (well, thread-local) state, meaning it affects all subsequent operations until the register is changed again. This means that to optimize any floating-point operation that might need rounding, the compiler has to statically predict what the value of the control register will be. That’s usually not going to be possible, so what compilers do instead is they just assume that the control register always remains in its default state. (Sometimes they provide ways to opt-out of that, but this is hard to do well and Rust currently has no facilities for this.) The status register is less obviously problematic, but note that if we say that a floating-point operation can mutate the status register, then it is no longer a pure operation, and therefore it cannot be freely reordered. To allow compilers to do basic optimizations like common subexpression elimination on floating-point operations, languages therefore generally also say that they consider the status register to be not observable.

What does this mean for inline assembly code that reads/writes these registers? For reading the status register, it means that the story code has no way of saying that this has anything to do with actual floating-point operations. The Abstract Machine has no floating-point status bits that the story code could read, so the best possible story is to return a non-deterministic value. This directly reflects the fact that the compiler makes no guarantees for the value that the program will observe in the status register, and since floating-point operations can be arbitrarily reordered, this should be taken quite literally.

For writing the control register, there simply is no possible story: no Rust operation exists that would change the rounding mode of subsequent floating-point operations. Any inline asm block that changes the rounding mode therefore has undefined behavior (and the same goes for other flags that change the behavior of instructions used by the Rust compiler, like flushing subnormals to zero).

While this may sound bleak, it is entirely possible to write an inline asm block that changes the rounding mode, performs some floating-point operations, and then changes it back! The story code for this block can use a soft-float library to perform exactly the same floating-point operations with a non-default rounding mode. Crucially, since the asm block overall leaves the control register unchanged, the story code does not even have to worry about that register. In other words, having a single big asm block that performs floating-point operations in a non-default rounding mode is fine. This also makes sense from an optimization perspective: there is no risk of the compiler moving a floating-point operation into the region of code where the rounding mode is different.

## Conclusion

I hope these examples were useful to demonstrate both the flexibility and limitations of the storytelling approach. In many cases, the inability to come up with a story directly correlates with potential miscompilations. This is great! Those are the kinds of inline asm blocks that we *have* to rule out as incorrect.[9](#fn:noopt) In some cases, however, there are no obvious miscompilations. And indeed, if we knew exactly which universal properties of Rust programs the compiler relies on, we could allow inline asm code that satisfies all those universal properties, even if it has no story which can be expressed in Rust source code. Unfortunately, this approach would require us to commit to the full set of universal properties the compiler may ever use. If we discover a new universal property tomorrow, we cannot use it since there might be an inline asm block for which the universal property does not hold.

This is why I am proposing to take the conservative approach: only allow inline asm blocks that are obviously compatible with all universal properties of actual Rust code, because their story can be expressed as actual Rust code. If there is an operation we want to allow that currently has no valid story, we should just [add](https://github.com/rust-lang/rfcs/pull/3700) a [new language operation](https://github.com/rust-lang/rfcs/pull/3605), which corresponds to officially blessing that operation as one the compiler will keep respecting.

Right now, we have no official documentation or guidelines for how inline asm blocks and FFI interact with Rust-level UB, but as the `innocent` example at the top of the post shows, we cannot leave inline asm blocks unconstrained like that. The storytelling approach is my proposal for filling that gap. I plan to eventually suggest it as the official rules for inline assembly. But before I do that, I’d like to be more confident that this approach really can handle most real-world scenarios. If you have examples of assembly blocks that cannot be explained with storytelling, but that you are convinced are correct and hence should be supported, please let us know, either in the immediate [discussion](https://www.reddit.com/r/rust/comments/1rshm93/how_to_use_storytelling_to_fit_inline_assembly/) for this blog post or (if you are reading this later) in the [t-opsem Zulip channel](https://rust-lang.zulipchat.com/#narrow/channel/136281-t-opsem).

#### Footnotes

1. FFI has one extra complication that does not arise with inline assembly, and that is cross-language LTO. That is its own separate can of worms and outside the scope of this post. [↩](#fnref:xlang)

2. The secret third option is that the program might be non-deterministic and allows both behaviors, but that definitely does not apply here. [↩](#fnref:nondet)

3. I can already sense that some people want to bring up CHERI as an apparent counterexample. CHERI has capabilities, which look and feel a bit like pointer provenance, but they are nowhere near fine-grained enough for Tree Borrows, so capabilities and provenance are still distinct concepts that should not be confused with each other. [↩](#fnref:cheri)

4. Credits go to Alice Ryhl for suggesting the term “telling a story” for this model. [↩](#fnref:alice)

5. Why am I insisting on volatile accesses here? Because if you had the page tables inside a regular Rust allocation, writes to that page table could have “interesting” effects, and that doesn’t really correspond to anything that can happen when you write to a normal Rust allocation. In other words, I haven’t (yet) come up with a proper story that would allow for those writes to be non-volatile. [↩](#fnref:volatile)

6. This assumes that we refine our specification of how memory allocation works in Rust so that there are regions of memory that “native” Rust allocations such as stack and static variables do not use, and that are instead entirely controlled by the program. If the only allocation operation a language has is “non-deterministic allocation anywhere in the address space”, this story does not work. [↩](#fnref:alloc-control)

7. Long-lived pointers into duplicated memory don’t really work because they might point to the wrong duplicate. But if that can be avoided, then you can just store them as integers and [cast them to a pointer](https://doc.rust-lang.org/std/ptr/fn.with_exposed_provenance.html) for every access; this avoids any long-lived provenance that might let the compiler apply normal allocation-based reasoning to this memory. [↩](#fnref:intptr)

8. All of the examples we found forgot to insert the `_mm_sfence`, which was clearly unsound. Thanks to the story, we now have a clear idea *why* it is unsound, i.e., which rule of the Rust language was violated. [↩](#fnref:forgotten-fence)

9. This assumes that we do not want to sacrifice the optimizations in question. Since inline assembly could hide inside any function call, this typically becomes a language-wide trade off: either we forbid such inline asm blocks, or we cannot do the optimization even in pure Rust code. [↩](#fnref:noopt)
