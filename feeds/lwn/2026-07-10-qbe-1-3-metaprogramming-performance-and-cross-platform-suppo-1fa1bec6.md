---
title: '[$] QBE 1.3: metaprogramming, performance, and cross-platform support'
url: https://lwn.net/Articles/1080519/
published: "2026-07-10T13:50:40Z"
feed: lwn
guid: https://lwn.net/Articles/1080519/
---

# [$] QBE 1.3: metaprogramming, performance, and cross-platform support

[QBE](https://c9x.me/compile/), a compact compiler backend developed by Quentin Carbonneaux, is a lightweight alternative to larger compiler backends such as LLVM and GCC. Designed to be small enough for a single developer to understand, QBE uses a [static single-assignment](https://en.wikipedia.org/wiki/Static_single-assignment_form) (SSA) intermediate representation (IR), supports the C ABI, and serves as the backend for projects such as [Hare](https://harelang.org/) and the [`cproc`](https://github.com/michaelforney/cproc) C11 compiler. Frontends emit the textual form of QBE's IR directly; QBE then takes care of register allocation, optimization, and native-code generation, producing assembly for the target architecture.
