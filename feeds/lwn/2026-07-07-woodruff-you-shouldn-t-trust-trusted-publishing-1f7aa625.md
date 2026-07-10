---
title: 'Woodruff: You shouldn''t trust trusted publishing'
url: https://lwn.net/Articles/1081690/
published: "2026-07-07T14:27:57Z"
feed: lwn
guid: https://lwn.net/Articles/1081690/
---

# Woodruff: You shouldn't trust trusted publishing

William Woodruff, better known online as "yossarian", has [published](https://blog.yossarian.net/2026/07/07/You-shouldnt-trust-trusted-publishing) a blog post to make the case that users should not place their trust in [trusted
publishing](https://docs.pypi.org/trusted-publishers/):

> Trusted Publishing is a mechanism for establishing trust between an external machine identity (like a CI/CD workflow) and one or more projects on a package index/registry. The "trust" in "Trusted Publishing" refers to that trust relationship, and not to anything else.
>
> It is not, and cannot be, a signal for package trust or quality. You cannot use it to determine whether a package is safe or "good," and PyPI consciously stymies attempts to misuse it for that purpose by not rendering it as a "green checkmark" or anything else of the sort.
>
> Or as another framing: Trusted Publishing is just a form of authentication. It doesn't tell you anything other than that an upload was authenticated, which all uploads to PyPI are.

LWN [covered](https://lwn.net/Articles/1076205/) trusted publishing in June.
