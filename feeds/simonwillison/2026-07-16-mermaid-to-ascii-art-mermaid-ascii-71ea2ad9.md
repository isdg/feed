---
title: Mermaid to ASCII art (mermaid-ascii)
url: https://simonwillison.net/2026/Jul/16/mermaid-ascii/#atom-everything
published: "2026-07-16T14:57:39Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jul/16/mermaid-ascii/#atom-everything
---

# Mermaid to ASCII art (mermaid-ascii)

**Tool:** [Mermaid to ASCII art (mermaid-ascii)](https://tools.simonwillison.net/mermaid-ascii)

After building the [Mermaid to ASCII tool based on Grok Build's Rust code](https://simonwillison.net/2026/Jul/16/grok-mermaid/) I learned that there's an older, more fully-featured Go library called [AlexanderGrooff/mermaid-ascii](https://github.com/AlexanderGrooff/mermaid-ascii) that implements a similar pattern, so I had Claude Fable 5 compile that one to WebAssembly as well so I could compare the two.

This one includes support for colors!

![Screenshot of a Mermaid diagram editor web app. A row of tab buttons reads: Flowchart, Multiple links, Subgraphs, Multi-line labels, Colors (selected, highlighted blue), Sequence, Alt fragment, Loop + note, Parallel. Below is a text input area containing: "graph LR / Build:::good --> Test:::good / Test --> Deploy:::warn / Deploy --> Rollback:::bad / classDef good color:#3fb950 / classDef warn color:#e3b341 / classDef bad color:#ff7b72". A control row shows an unchecked "ASCII only" checkbox, "Padding X: 5", "Padding Y: 5", "Box padding: 1", and buttons "Copy as text" and "Copy link to this diagram". At the bottom on a black background is the rendered left-to-right flowchart with four connected boxes: "Build" (green text), "Test" (green text), "Deploy" (yellow text), "Rollback" (red text), each linked by arrows.](https://static.simonwillison.net/static/2026/mermaid-ascii.webp)

Tags: [go](https://simonwillison.net/tags/go), [tools](https://simonwillison.net/tags/tools), [webassembly](https://simonwillison.net/tags/webassembly), [mermaid](https://simonwillison.net/tags/mermaid)
