---
title: '[$] Limiting negative dentries'
url: https://lwn.net/Articles/1079407/
published: "2026-07-03T14:10:03Z"
feed: lwn
guid: https://lwn.net/Articles/1079407/
---

# [$] Limiting negative dentries

A number of problems related to negative directory entries (dentries) were the topic of a filesystem-track session at the 2026 [Linux Storage,
Filesystem, Memory Management, and BPF Summit](https://events.linuxfoundation.org/lsfmmbpf/). Negative dentries are used to indicate that a file of a given name does not exist in a directory; it is an optimization that short-circuits the lookup of the file name when the answer is already known.

Miklos Szeredi led a session that discussed some problems that come from having too many negative dentries for a directory.
