---
title: Announcing Rust 1.96.1
url: https://blog.rust-lang.org/2026/06/30/Rust-1.96.1/
published: "2026-06-30T00:00:00Z"
feed: rust-lang
guid: https://blog.rust-lang.org/2026/06/30/Rust-1.96.1/
---

# Announcing Rust 1.96.1

The Rust team has published a new point release of Rust, 1.96.1. Rust is a programming language that is empowering everyone to build reliable and efficient software.

If you have a previous version of Rust installed via rustup, getting Rust 1.96.1 is as easy as:

```
rustup update stable
```

If you don't have it already, you can [get `rustup`](https://www.rust-lang.org/install.html) from the appropriate page on our website.

## What's in 1.96.1

Rust 1.96.1 fixes:

- [Missing retries / timeouts in Cargo's HTTP client](https://github.com/rust-lang/cargo/pull/17131)
- [Miscompilation in a MIR optimization](https://github.com/rust-lang/rust/pull/158214)

It also [fixes](https://github.com/rust-lang/cargo/pull/17140) three CVEs affecting libssh2 (which is compiled into Cargo):

- [CVE-2025-15661](https://www.cve.org/CVERecord?id=CVE-2025-15661)
- [CVE-2026-55199](https://www.cve.org/CVERecord?id=CVE-2026-55199)
- [CVE-2026-55200](https://www.cve.org/CVERecord?id=CVE-2026-55200)

### Contributors to 1.96.1

Many people came together to create Rust 1.96.1. We couldn't have done it without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.96.1/)
