---
title: '[$] The kernel''s iomap layer'
url: https://lwn.net/Articles/1079415/
published: "2026-07-06T14:37:34Z"
feed: lwn
guid: https://lwn.net/Articles/1079415/
---

# [$] The kernel's iomap layer

Conversations about the kernel's filesystem implementations often involve a layer called "iomap", but relatively few people can reliably say what iomap actually is. That is just the kind of gap that LWN exists to fill. In short, iomap handles the mapping between data in the filesystem space (identified by a file of interest, and an offset within that file) and in the storage space (which may be a memory location, or a set of blocks on a storage device). Using that mapping, iomap handles a long list of common, filesystem-related tasks, allowing a lot of boilerplate code to be removed from individual filesystem implementations.
