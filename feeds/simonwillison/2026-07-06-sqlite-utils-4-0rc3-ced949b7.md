---
title: sqlite-utils 4.0rc3
url: https://simonwillison.net/2026/Jul/6/sqlite-utils/#atom-everything
published: "2026-07-06T05:40:08Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jul/6/sqlite-utils/#atom-everything
---

# sqlite-utils 4.0rc3

**Release:** [sqlite-utils 4.0rc3](https://github.com/simonw/sqlite-utils/releases/tag/4.0rc3)

I hoped to release `sqlite-utils 4.0` stable this weekend, but as I worked through the backlog of issues and PRs with a combination of Claude Fable 5 and GPT-5.5 the changelog since rc2 [kept getting bigger](https://sqlite-utils.datasette.io/en/latest/changelog.html#rc3-2026-07-05).

The biggest new feature is support for introspecting and creating compound foreign keys - a feature that involves a subtle breaking change to [table.foreign\_keys](https://sqlite-utils.datasette.io/en/latest/python-api.html#foreign-keys) and hence needed to land for the 4.0 stable release.

`sqlite-utils` also now follows SQLite's convention for case insensitive column names, which turned out to touch [a bunch of different places at once](https://sqlite-utils.datasette.io/en/latest/changelog.html#case-insensitive-column-matching).

Tags: [projects](https://simonwillison.net/tags/projects), [sqlite](https://simonwillison.net/tags/sqlite), [sqlite-utils](https://simonwillison.net/tags/sqlite-utils), [annotated-release-notes](https://simonwillison.net/tags/annotated-release-notes), [gpt](https://simonwillison.net/tags/gpt), [claude-mythos-fable](https://simonwillison.net/tags/claude-mythos-fable)
