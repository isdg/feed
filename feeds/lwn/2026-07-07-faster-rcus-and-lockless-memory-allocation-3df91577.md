---
title: '[$] Faster RCUs and lockless memory allocation'
url: https://lwn.net/Articles/1081009/
published: "2026-07-07T13:39:34Z"
feed: lwn
guid: https://lwn.net/Articles/1081009/
---

# [$] Faster RCUs and lockless memory allocation

Puranjay Mohan shared some of the [work](https://lwn.net/ml/all/20260417231203.785172-1-puranjay@kernel.org/) he's been doing recently on improving the performance of read-copy-update (RCU) at the 2026 [Linux
Storage, Filesystem, Memory-Management, and BPF Summit](https://events.linuxfoundation.org/lsfmmbpf/); his talk would have been nice context to have earlier in the day when Harry Yoo and Alexei Starovoitov led a session about the [new `kmalloc_nolock()` function](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=af92793e52c3) that allows for lockless allocation from any kernel context, and which interacts with the RCU subsystem to allow that. This article therefore covers the two sessions together and in the reverse order, to provide that missing context.
