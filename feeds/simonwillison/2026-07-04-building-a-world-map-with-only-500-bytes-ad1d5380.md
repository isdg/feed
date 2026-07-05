---
title: Building a World Map with only 500 bytes
url: https://simonwillison.net/2026/Jul/4/building-a-world-map-with-only-500-bytes/#atom-everything
published: "2026-07-04T23:09:02Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jul/4/building-a-world-map-with-only-500-bytes/#atom-everything
---

# Building a World Map with only 500 bytes

**[Building a World Map with only 500 bytes](https://www.experimentlog.com/blog/building-a-world-map-with-only-500-bytes)**

Iwo Kadziela (assisted by Codex) figured out a way to generate a credible ASCII world map using 445 bytes of data:

![A map of the world rendered as black asterisk ASCII characters, it looks very good](https://static.simonwillison.net/static/2026/world-map-ascii.png)

The key trick is to use deflate compression, which is then wired together using this neat snippet of JavaScript. I didn't know you could use `fetch()` with `data:` URIs like this:

```
fetch('data:;base64,1ZpLsgIxCEXnrM...==').then(
  r => r.body.pipeThrough(new DecompressionStream('deflate-raw'))
).then(
  s => new Response(s).text()
).then(
  t => b.innerHTML = '<pre style=font-size:.65vw>' + t
)

```

Via [Hacker News](https://news.ycombinator.com/item?id=48747762)

Tags: [ascii-art](https://simonwillison.net/tags/ascii-art), [data-urls](https://simonwillison.net/tags/data-urls), [javascript](https://simonwillison.net/tags/javascript)
