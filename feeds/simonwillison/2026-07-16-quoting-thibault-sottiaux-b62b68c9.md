---
title: Quoting Thibault Sottiaux
url: https://simonwillison.net/2026/Jul/16/bad-codex-bug/#atom-everything
published: "2026-07-16T17:45:59Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jul/16/bad-codex-bug/#atom-everything
---

# Quoting Thibault Sottiaux

> On file deletions. We’ve investigated a handful of reports where GPT-5.6 unexpectedly deleted files.
>
> What we have found is that this most commonly occurs when:
> - Full access mode is enabled and codex is run without sandboxing protections, including without auto review being enabled
> - The model attempts to override the $HOME env var to define a temporary directory.
> - The model makes an honest mistake and mistakenly deletes $HOME instead.

— [Thibault Sottiaux](https://twitter.com/thsottiaux/status/2077630111499882637), describing a pretty gnarly Codex bug

Tags: [codex](https://simonwillison.net/tags/codex), [coding-agents](https://simonwillison.net/tags/coding-agents), [generative-ai](https://simonwillison.net/tags/generative-ai), [ai](https://simonwillison.net/tags/ai), [llms](https://simonwillison.net/tags/llms)
