---
title: Many old shim versions are still accepted by secure boot
url: https://lwn.net/Articles/1082940/
published: "2026-07-15T12:49:36Z"
feed: lwn
guid: https://lwn.net/Articles/1082940/
---

# Many old shim versions are still accepted by secure boot

The CMU CERT Coordination Center has put out [an advisory](https://kb.cert.org/vuls/id/616257) that many exploitable versions of the shim binary, used to boot Linux on systems with UEFI secure boot enabled, were never added to the revocation list.

> An attacker with administrative privileges or the ability to modify the boot process could use one of the vulnerable shim bootloaders to bypass Secure Boot protections and execute arbitrary code before the operating system loads. Code executed during this early boot phase may achieve persistent compromise of the platform, including the ability to load unsigned or malicious kernel components that can survive system reboots and, in some cases, operating system reinstallation.

The advisory contains a list of vulnerable shims.
