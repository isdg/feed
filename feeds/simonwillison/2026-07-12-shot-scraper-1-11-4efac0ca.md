---
title: shot-scraper 1.11
url: https://simonwillison.net/2026/Jul/12/shot-scraper/#atom-everything
published: "2026-07-12T23:46:52Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jul/12/shot-scraper/#atom-everything
---

# shot-scraper 1.11

**Release:** [shot-scraper 1.11](https://github.com/simonw/shot-scraper/releases/tag/1.11)

Some minor improvements, mainly around command option consistency and making the `server:` mechanism [used by](https://shot-scraper.datasette.io/en/stable/multi.html#running-a-server-for-the-duration-of-the-session) both `shot-scraper video` and `shot-scraper multi` work if the server takes longer than a second to start serving traffic.

> - `server:` processes used by `shot-scraper multi` and `shot-scraper video` now wait up to 30 seconds for the target URL to accept connections, polling for port availability and replacing the previous fixed one-second delay. [#197](https://github.com/simonw/shot-scraper/issues/197)
> - The `shot-scraper`, `pdf`, `html`, `accessibility` and `har` commands now have a `--js-file` option for loading JavaScript from a local file, standard input or `gh:username/script`, as an alternative to `--javascript` which accepts the string of JavaScript directly as an argument. [#192](https://github.com/simonw/shot-scraper/issues/192)
> - `shot-scraper multi` supports the equivalent `js_file:` YAML key.
> - The `shot-scraper javascript` and `shot-scraper html` commands now have a `--timeout` option for consistency with other commands. [#118](https://github.com/simonw/shot-scraper/issues/118)

Tags: [shot-scraper](https://simonwillison.net/tags/shot-scraper)
