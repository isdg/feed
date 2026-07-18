---
title: Building an Arch Linux aarch64 port for Holo Core (Collabora blog)
url: https://lwn.net/Articles/1083392/
published: "2026-07-17T17:03:50Z"
feed: lwn
guid: https://lwn.net/Articles/1083392/
---

# Building an Arch Linux aarch64 port for Holo Core (Collabora blog)

Collabora has published a [blog
post](https://www.collabora.com/news-and-blog/news-and-events/building-an-arch-linux-aarch64-port-for-holo-core.html) about its work with Valve on Holo Core, which is a port of Arch Linux to aarch64 to be used as the the operating system on Valve's 64-bit Arm Steam Frame gaming system. Collabora has released the [sources](https://gitlab.steamos.cloud/holo/holo-core-aarch64-preview), [binary
packages](https://steamdeck-packages.steamos.cloud/holo-core-aarch64-preview/mash-20251118.3/), and a container image for aarch64 devices. The post describes some of the challenges in porting Arch Linux to a new architecture, and what remains to be done:

> Whilst the infrastructure developed to this point is capable of building from first principles up until a point-in-time snapshot, the next step is to build this into a system which can track Arch Linux as it is developed. This work will serve as the basis of a continuously-operating CI system capable of shadowing Arch Linux itself. We will work with the upstream Arch Linux project to help Arch with their efforts to port the distribution to `aarch64` architecture and work towards automated repeatable builds.

The post also includes instructions on how to create and test an aarch64 build container on an x86\_64 host, for users who would like to follow along at home but lack a 64-bit Arm device.
