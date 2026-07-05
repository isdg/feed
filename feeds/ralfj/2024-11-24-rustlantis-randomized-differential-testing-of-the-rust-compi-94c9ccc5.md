---
title: 'Rustlantis: Randomized Differential Testing of the Rust Compiler'
url: https://www.ralfj.de/blog/2024/11/25/rustlantis.html
published: "2024-11-24T23:00:00Z"
feed: ralfj
guid: https://www.ralfj.de/blog/2024/11/25/rustlantis.html
---

# Rustlantis: Randomized Differential Testing of the Rust Compiler

The first paper produced entirely by my group has recently been published at OOPSLA. :) The paper is about fuzzing the optimizations and code generation of the Rust compiler by randomly generating MIR programs and ensuring they behave the same across different backends, different optimization levels, and in Miri. The core part of this work was done by Andy (Qian Wang) for his [master thesis](https://ethz.ch/content/dam/ethz/special-interest/infk/inst-pls/plf-dam/documents/StudentProjects/MasterTheses/2023-Andy-Thesis.pdf). This was already a strong thesis, but Andy kept working on this even after he started having a regular dayjob, and we ended up with a very nice paper. In total, he found 22 new bugs in the Rust compiler, 12 of them in the LLVM backend that has already been extensively fuzzed by prior work.

To learn more, [check out the paper](https://plf.inf.ethz.ch/research/oopsla24-rustlantis.html) or [watch Andy’s talk](https://www.youtube.com/watch?v=kHYEHSHLffU&t=20447s) (the timestamp link seems unreliable, seek to the 5h40min mark if it doesn’t do that automatically).
