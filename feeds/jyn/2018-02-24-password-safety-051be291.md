---
title: Password Safety
url: https://jyn.dev/password-safety/
published: "2018-02-24T00:00:00Z"
feed: jyn
guid: https://jyn.dev/password-safety/
---

# Password Safety

On the web today, there many different services we use. We use email, Facebook, GitHub, Google, Twitter, dozens of various services. Each of these require their own username and password.

We're told, of course, to use a different password for each account. [Only 70% of us](https://www.troyhunt.com/science-of-password-selection/), however, actually do so. How are we supposed to remember a [dozen random passwords](https://www.nist.gov/publications/character-strings-memory-and-passwords-what-recall-study-can-tell-us), each different? Traditionally, we use the same password for each, perhaps with [slight differences](https://reusablesec.blogspot.com/2010/10/new-paper-on-password-security-metrics.html). This makes it very easy for hackers to [get into your accounts](https://xkcd.com/792/).

There are very few common remedies for this - there's ways to [check if you've been hacked](https://haveibeenpwned.com/) and [multi-factor auth](https://en.wikipedia.org/wiki/Multi-factor_authentication), but most peoples' response is to either write sticky notes of passwords, or ignore the problem altogether.

This does work if you don't [leave your passwords lying around](https://jyn.dev/assets/password.jpg). It is inconvenient, however, and doesn't scale to a large password database. What you should be using is a [password manager](https://en.wikipedia.org/wiki/Password_manager).

## Password Managers [Anchor link for: password-managers](https://jyn.dev/password-safety/\#password-managers)

A password manager is an easy way to keep track of passwords. It can be anything from an [encrypted Excel Spreadsheet](https://support.office.com/en-us/article/Protect-an-Excel-file-7359d4ae-7213-4ac2-b058-f75e9311b599) to [your browser's builtin manager](https://duckduckgo.com/?q=browser+manage+password+saving). It takes one 'master password', and in return keeps track of everything else for you. There's no need to remember multiple passwords, it's easy to meet any complexity requirement, and it doesn't matter how often you're required to change your passwords (which is [bad practice](http://cs.unc.edu/~fabian/papers/PasswordExpire.pdf), by the way.

## What to use [Anchor link for: what-to-use](https://jyn.dev/password-safety/\#what-to-use)

Wikipedia has [an extensive list](https://en.wikipedia.org/wiki/List_of_password_managers) of password managers.

When choosing a manager, you should consider several things:

- What features are you looking for? What would be a dealbreaker?
- How much security are you looking for? What is your [threat model](https://en.wikipedia.org/wiki/Threat_model)?
- How much convenience are you willing to sacrifice?

I recommend [Keepass](https://keepass.info) because it

- encrypts your databases with [state of the art encryption](https://keepass.info/help/base/security.html)
- organizes your passwords [by groups](https://keepass.info/features.html#lnkgroups)
- stores usernames, URLs, and [arbitrary data](https://keepass.info/features.html#lnktimes)
- randomly [generates passwords](https://keepass.info/help/base/pwgenerator.html)
- is [open source](https://keepass.info/download.html)
- is fully interoperable with many [other programs](https://en.wikipedia.org/wiki/KeePass#Unofficial_KeePass_derivatives)
- has [many other features](https://keepass.info/features.html)

Once you start using a password manager, you can start using passwords like `W(xv|u7N''fAs,{t|J` for all your accounts.

## Appendix [Anchor link for: appendix](https://jyn.dev/password-safety/\#appendix)

- If you're not on Windows, or just want more features, check out [Keepassxc](https://keepassxc.org/) (Keepass Cross-Platform Community Edition). It supports global autotype and [timed one-time passwords](https://en.wikipedia.org/wiki/Time-based_One-time_Password_Algorithm), a.k.a. two-factor auth.
- I've written a [presentation on secure passwords](https://drive.google.com/open?id=15u4uXxC5K7v2Llsu8L4JVDcFNjyavW4pKBJ2GW5YV5M)
- [StackOverflow on password reuse](https://security.stackexchange.com/q/6682)
- The figure for reuse [may be lower](https://www.lightbluetouchpaper.org/2011/02/09/measuring-password-re-use-empirically/) than I've claimed, but it's still a minimum of 30%
