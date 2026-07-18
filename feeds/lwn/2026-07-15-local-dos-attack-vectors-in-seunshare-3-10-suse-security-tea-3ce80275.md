---
title: Local DoS attack vectors in seunshare 3.10 (SUSE Security Team Blog)
url: https://lwn.net/Articles/1083076/
published: "2026-07-15T15:52:13Z"
feed: lwn
guid: https://lwn.net/Articles/1083076/
---

# Local DoS attack vectors in seunshare 3.10 (SUSE Security Team Blog)

The SUSE Security Team Blog has a [post](https://security.opensuse.org/2026/07/15/selinux-seunshare.html) with an analysis of [`seunshare`](https://man7.org/linux/man-pages/man8/seunshare.8.html), which is used by SELinux to confine untrusted programs. During a review of [version
3.10](https://github.com/SELinuxProject/selinux/releases/tag/3.10) of the program, the team identified two local Denial-of-Service (DoS) vectors.

> Since `seunshare` is supposed to run on SELinux-enabled systems, it is important to understand what kind of privilege escalation can be achieved when vulnerabilities are exploited in a setuid-root binary like this. Many SELinux-enabled systems, such as Fedora and openSUSE, ship with the "targeted" SELinux policy by default. This policy is focused on confining well-known system services, but assigns an unconfined SELinux context to interactive users by default to achieve a balance between security and usability.
>
> There is currently no domain transition from the unconfined domain to the more restricted `seunshare_t` defined in the SELinux policy for `seunshare`. This means the execution of `seunshare` continues in the unconfined domain. Thus in the context of attacks carried out by interactive users, the impact of the vulnerabilities below will be a root-like privilege escalation despite the system running in SELinux enforced mode.

See the post for the full write-up of the team's discoveries and timeline. The vulnerabilities have been fixed in [version 3.11](https://github.com/SELinuxProject/selinux/releases/tag/3.11).
