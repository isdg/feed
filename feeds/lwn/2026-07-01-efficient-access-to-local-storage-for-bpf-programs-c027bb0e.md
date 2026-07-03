---
title: '[$] Efficient access to local storage for BPF programs'
url: https://lwn.net/Articles/1078968/
published: "2026-07-01T17:07:42Z"
feed: lwn
guid: https://lwn.net/Articles/1078968/
---

# [$] Efficient access to local storage for BPF programs

When a BPF program is used to filter or redirect packets in the networking subsystem, the program will often want to associate data with each packet as it moves through the kernel. The kernel's [local BPF storage API](https://docs.ebpf.io/linux/helper-function/bpf_get_local_storage/), which associates extra data with some kernel objects, provides a way to do that. (See also the [BPF map types](https://docs.kernel.org/bpf/maps.html) that end in `STORAGE`.) Amery Hung and Jakub Sitnicki led two sessions at the 2026 [Linux Storage, Filesystem, Memory-Management, and BPF Summit](https://events.linuxfoundation.org/lsfmmbpf/) about how to make accesses to local storage data more efficient. Hung spoke about general performance problems related to locking, while Sitnicki examined the use of local storage in the networking subsystem in particular.
