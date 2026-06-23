---
title: '[$] KASAN for JIT-compiled BPF code'
url: https://lwn.net/Articles/1077740/
published: "2026-06-23T15:53:01Z"
feed: lwn
guid: https://lwn.net/Articles/1077740/
---

# [$] KASAN for JIT-compiled BPF code

Alexis Lothoré has been working to add support for the kernel's memory-access checker, [KASAN](https://docs.kernel.org/dev-tools/kasan.html), to just-in-time-compiled BPF code. He spoke about that work at the 2026 [Linux Storage, Filesystem, Memory-Management, and BPF Summit](https://events.linuxfoundation.org/lsfmmbpf/). KASAN support is needed, he said, to help catch bugs in the BPF just-in-time (JIT) compiler. KASAN is a great tool for catching memory-management problems in the kernel, but only in code that can be monitored by it.
