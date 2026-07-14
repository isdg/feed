---
title: '[$] Sending packets directly from BPF'
url: https://lwn.net/Articles/1081696/
published: "2026-07-14T13:16:52Z"
feed: lwn
guid: https://lwn.net/Articles/1081696/
---

# [$] Sending packets directly from BPF

[Tetragon](https://tetragon.io/), the BPF-based security monitoring tool, uses BPF to monitor different aspects of a running kernel and enforce user-specified policies. It sends its data to a user-space process, which forwards the data to a central monitoring service elsewhere in the network, however. This presents a point of vulnerability: if an attacker can kill Tetragon's user-space agent, it won't be able to properly report on the situation. Song Liu, Mahé Tardy, and Liam Wiseheart spoke about their work removing the need for the user-space agent at the 2026 [Linux Storage, Filesystem, Memory-Management, and
BPF Summit](https://events.linuxfoundation.org/lsfmmbpf/).
