---
title: Scheduling Do-Not-Disturb in GNOME
url: https://jyn.dev/do-not-disturb-in-gnome/
published: "2025-02-22T00:00:00Z"
feed: jyn
guid: https://jyn.dev/do-not-disturb-in-gnome/
---

# Scheduling Do-Not-Disturb in GNOME

## Do Not Disturb [Anchor link for: do-not-disturb](https://jyn.dev/do-not-disturb-in-gnome/\#do-not-disturb)

GNOME has a little button that lets you turn on Do-Not-Disturb for notifications:

![Gnome notifications menu](https://jyn.dev/assets/Pasted%20image%2020250222135047.png)

Unfortunately, it has [no way of scheduling DnD](https://gitlab.gnome.org/GNOME/gnome-control-center/-/issues/2200).

Good news, though! It does support turning on DnD through the CLI: `gsettings set org.gnome.desktop.notifications show-banners false`. I put that in a script named `toggle-dnd` in my dotfiles:

```sh
$ cat bin/toggle-dnd
#!/bin/sh

case ${1:-} in
	# "show-banners" is reversed from what you would expect "do not disturb" to mean
	true) new=false;;
	false) new=true;;
	*)
	if [ "$(gsettings get org.gnome.desktop.notifications show-banners)" = true ]
		then new=false
		else new=true
	fi
	;;
esac
gsettings set org.gnome.desktop.notifications show-banners $new

```

## scheduling [Anchor link for: scheduling](https://jyn.dev/do-not-disturb-in-gnome/\#scheduling)

I tried putting that in cron[1](https://jyn.dev/do-not-disturb-in-gnome/#fn-1), had a sneaking suspicion it wouldn't work, set it to run every minute, and saw this very unhelpful line of logging:

```console
$ journalctl --unit cron --since '5m ago'
Feb 22 12:00:01 pop-os CRON[1623131]: (CRON) info (No MTA installed, discarding output)

```

Ok, fine. Let's pipe the output to the system log[2](https://jyn.dev/do-not-disturb-in-gnome/#fn-2), since clearly cron can't handle that itself.

```crontab
* * * * * bash -lc 'org.gnome.desktop.notifications show-banners false 2>&1 | logger -t toggle-dnd'

```

That at least shows more useful output.

```console
$ journalctl -t toggle-dnd
Feb 22 05:59:01 pop-os toggle-dnd[1376772]: /bin/sh: 1: toggle-dnd: not found

```

Oh right. Cron is running things with a default PATH. Technically there are ways to configure this [3](https://jyn.dev/do-not-disturb-in-gnome/#fn-3), but the simple solution is just to run a bash login shell which sources all the directories i would normally have in a shell. at this point, however, it is getting somewhat annoying to test via cron, so let's replicate cron's environment:

```console
$ env -i HOME="$HOME" TERM="$TERM" PS1='$ ' HISTSIZE=-1 HISTFILE= bash --norc --noprofile
$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
$ bash -l
$ echo $PATH
/home/jyn/Documents/node-v20.12.2-linux-x64/bin:/home/jyn/.local/bin:/home/jyn/src/dotfiles/bin:/home/jyn/.local/lib/cargo/bin:/snap/bin:/usr/games:/home/jyn/perl5/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin

```

neat. let's run make sure our command runs. before:

![Left side of the Gnome topbar](https://jyn.dev/assets/Pasted%20image%2020250222141930.png)

after running `toggle-dnd` in our login shell:

![Still the left side of the gnome topbar. In fact this is the exact same image as before.](https://jyn.dev/assets/Pasted%20image%2020250222141930.png)

... nothing happened. we can confirm this on the CLI:

```console
$ gsettings get org.gnome.desktop.notifications show-banners
true
$ gsettings set org.gnome.desktop.notifications show-banners false
$ gsettings get org.gnome.desktop.notifications show-banners
true

```

## DBUS [Anchor link for: dbus](https://jyn.dev/do-not-disturb-in-gnome/\#dbus)

at this point i started to get annoyed and ran `systemctl --user status` in hopes of writing a systemd timer instead. fortunately, i did that inside the bash login shell, which gave me this helpful error message:

```console
$ systemctl --user status
Failed to connect to bus: $DBUS_SESSION_BUS_ADDRESS and $XDG_RUNTIME_DIR not defined (consider using --machine=<user>@.host --user to connect to bus of other user)

```

It turns out that both `gsettings` and `systemctl` are trying to communicate over DBUS, and DBUS is linked to your "user session", set when you login. Unsetting environment variables disables DBUS [4](https://jyn.dev/do-not-disturb-in-gnome/#fn-4).

I found a helpful [stackoverflow post](https://askubuntu.com/a/1468012) that helps us reconnect to DBUS [5](https://jyn.dev/do-not-disturb-in-gnome/#fn-5):

```sh
export XDG_RUNTIME_DIR="/run/user/$UID"
export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"

```

Let's write a little abstraction for that, too [6](https://jyn.dev/do-not-disturb-in-gnome/#fn-6).

```bash
#!/usr/bin/env bash
# Run a command in an environment where DBUS commands (e.g. `systemd --user`, `gsettings`) are available
: "${XDG_RUNTIME_DIR:="/run/user/$UID"}"
: "${DBUS_SESSION_BUS_ADDRESS:="unix:path=${XDG_RUNTIME_DIR}/bus"}"
export XDG_RUNTIME_DIR DBUS_SESSION_BUS_ADDRESS
exec "$@"

```

Now, finally, we can put the pieces together:

```console
$ crontab -l
0 20 * * * bash -lc 'dbus-run-user toggle-dnd true  2>&1 | logger -t toggle-dnd'
0 8  * * * bash -lc 'dbus-run-user toggle-dnd false 2>&1 | logger -t toggle-dnd'

```

### P.S. [Anchor link for: p-s](https://jyn.dev/do-not-disturb-in-gnome/\#p-s)

#### X11 [Anchor link for: x11](https://jyn.dev/do-not-disturb-in-gnome/\#x11)

just setting up DBUS doesn't set up our X11 environment again. I didn't happen to need that. but now that we have DBUS, you can retrieve it pretty easily:

```console
$ dbus-run-user systemctl --user show-environment | grep ^DISPLAY
DISPLAY=:1

```

You can do something similar for `XAUTHORITY`, `TMUX*`, and `XDG_*` environment variables. Note that `systemctl --user` may be using a tmux session or pane that no longer exists; use caution.

#### why were you doing this in the first place [Anchor link for: why-were-you-doing-this-in-the-first-place](https://jyn.dev/do-not-disturb-in-gnome/\#why-were-you-doing-this-in-the-first-place)

hahahaha so i had the foolish idea that this would get discord to silence notifications at night. [it does not do that](https://support.discord.com/hc/en/community/posts/22549582088343-Respect-desktop-s-Do-Not-Disturb-mode-for-desktop-notifications). i ended up just turning off desktop notification sounds altogether.

1. "why not a systemd user timer" because i don't know a helper that lets you write the timer schedule interactively, the way that [https://cron.help/](https://cron.help/) works for crontabs, and because systemd's documentation is smeared across a ton of different man pages. cron is just `crontab -e`. [↩](https://jyn.dev/do-not-disturb-in-gnome/#fr-1-1)

2. "why not set cron to automatically write output to the system log" hahahahaha there's no way to do that. unless you're using specifically [`cronie`](https://man.archlinux.org/man/extra/cronie/cron.8.en#OPTIONS), which isn't packaged in Ubuntu 22.04. [↩](https://jyn.dev/do-not-disturb-in-gnome/#fr-2-1)

3. see [`man 5 crontab`](https://manpages.ubuntu.com/manpages/trusty/man5/crontab.5.html); also this differs depending which version of cron you have installed. this link for instance does not match the man page i have installed locally for `man 5 crontab`. [↩](https://jyn.dev/do-not-disturb-in-gnome/#fr-3-1)

4. it's almost like [using environment variables to communicate data through a system is a bad idea](https://blog.sunfishcode.online/no-ghosts/)! [↩](https://jyn.dev/do-not-disturb-in-gnome/#fr-4-1)

5. note that recommends using `machinectl` or `systemctl --machine=$USER@localhost --user` instead. but both of those don't work in this environment: `machinectl` requires interactive login, and `--machine` just doesn't work at all: `Failed to connect to bus: Host is down` `Failed to list units: Transport endpoint is not connected` [↩](https://jyn.dev/do-not-disturb-in-gnome/#fr-5-1)

6. This uses bash because that sets `$UID` for us automatically. Theoretically we could do this with sh and `id -u`, but it's more of a pain than it's worth. [↩](https://jyn.dev/do-not-disturb-in-gnome/#fr-6-1)
