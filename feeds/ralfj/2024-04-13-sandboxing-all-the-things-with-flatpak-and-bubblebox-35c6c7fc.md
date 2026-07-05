---
title: Sandboxing All The Things with Flatpak and BubbleBox
url: https://www.ralfj.de/blog/2024/04/14/bubblebox.html
published: "2024-04-13T22:00:00Z"
feed: ralfj
guid: https://www.ralfj.de/blog/2024/04/14/bubblebox.html
---

# Sandboxing All The Things with Flatpak and BubbleBox

A few years ago, I have [blogged](/blog/2019/03/09/firejail.html) about my approach to sandboxing less-trusted applications that I have to or want to run on my main machine. The approach has changed since then, so it is time for an update.

Over time I grew increasingly frustrated with Firejail: configurations would frequently break on updates, and debugging Firejail profiles is extremely hard. When considering all the included files, we are talking about many hundred lines of configuration with a subtle interplay of allowlists and blocklists. Even when I knew which folder I wanted to give access to, it was often non-trivial to ensure that this access would actually be possible.

Now I am instead using a combination of two different approaches: Flatpak and BubbleBox.

## Flatpak

The easiest sandbox to maintain is the sandbox maintained by someone else. So when a Flatpak exists for software I want to or have to use, such as Signal or Zoom, that is generally my preferred approach.

Unfortunately, Flatpaks can come with extremely liberal default profiles that make the sandbox mostly pointless. The following global overrides help ensure that this does not happen:

```
[Context]
sockets=!gpg-agent;!pcsc;!ssh-auth;!system-bus;!session-bus
filesystems=~/.XCompose:ro;xdg-config/fontconfig:ro;!~/.gnupg;!~/.ssh;!xdg-documents;!home;!host

[Session Bus Policy]
org.freedesktop.Flatpak=none
org.freedesktop.secrets=none

```

I also use [Flatseal](https://flathub.org/apps/com.github.tchx84.Flatseal), an amazing application that helps to check which permissions applications get, and change them if necessary.

## BubbleBox

However, not all software exists as Flatpak. Also, sometimes I want software to run basically on my host system (i.e., to use the regular `/usr`), just without access to literally *everything* in my home directory. Examples of this are Factorio and VSCodium. The latter doesn’t work in Flatpak as I want to use it with LaTeX, and realistically this means it needs to run the LaTeX installed via `apt`. The official recommendation is to effectively disable the Flatpak sandbox, but that entirely defeats the point, so I went looking for alternatives.

[bubblewrap](https://github.com/containers/bubblewrap) provides a very convenient solution: it can start an application in its own private filesystem namespace with full control over which part of the host file system is accessible from inside the sandbox. I wrote a small wrapper around bubblewrap to make this configuration a bit more convenient to write and manage; this project is called [BubbleBox](/projects/bubblebox). This week-end I finally got around to adding support for [xdg-dbus-proxy](https://github.com/flatpak/xdg-dbus-proxy) so that sandboxed applications can now access particular D-Bus functions without having access to the entire bus (which is in general not safe to expose to a sandboxed application). That means it’s finally time to blog about this project, so here we go – if you are interested, check out [BubbleBox](/projects/bubblebox); the project page explains how you can use it to set up your own sandboxing.[1](#fn:1)

I should also note that this is not the only bubblewrap-based sandboxing solution. [bubblejail](https://github.com/igo95862/bubblejail) is fairly similar but provides a configuration GUI and a good set of default provides; it was a very useful resource when figuring out the right bubblewrap flags to make complex GUI applications work properly. (Incidentally, “bubblejail” is also how I called my own script originally, but then I realized that the name is already taken.) Joachim Breitner also recently [blogged](https://www.joachim-breitner.de/blog/812-Convenient_sandboxed_development_environment) about his own bubblewrap-based sandboxing script. sloonz has a similar [script](https://gist.github.com/sloonz/4b7f5f575a96b6fe338534dbc2480a5d) as well, with a nice yaml-based configuration format and [great explanations](https://sloonz.github.io/posts/sandboxing-1/) for what all the flags exactly do. Had their script existed when I started what eventually became BubbleBox, I would have used it as a starting point. But it was also fun to figure out my own solution.

Using bubblewrap and xdg-dbus-proxy for this was an absolute joy. Both of these components came out of the Flatpak project, but the authors realized that they could be independently useful, so in best Unix tradition they turned them into tools that provide all the required mechanism without hard-coding any sort of policy. Despite doing highly non-trivial tasks, they are both pretty easy to use and compose and very well-documented. Thanks a lot to everyone involved!

1. One day I should probably rewrite this in Rust… maybe this will be my test project for when [cargo-script](https://rust-lang.github.io/rfcs/3424-cargo-script.html) becomes available. [↩](#fnref:1)
