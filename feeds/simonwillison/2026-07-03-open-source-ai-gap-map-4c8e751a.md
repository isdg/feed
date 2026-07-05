---
title: Open Source AI Gap Map
url: https://simonwillison.net/2026/Jul/3/open-source-ai-gap-map/#atom-everything
published: "2026-07-03T22:04:31Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jul/3/open-source-ai-gap-map/#atom-everything
---

# Open Source AI Gap Map

**[Open Source AI Gap Map](https://map.currentai.org)**

[Current AI](https://www.currentai.org) is "a global partnership building a public option for AI", founded as a non-profit at the AI Action Summit in Paris in February 2025 and backed by serious capital ($400m already committed).

They [launched their Gap Map](https://www.currentai.org/blogs/introducing-the-gap-map-v0-1) a couple of days ago - an attempt at indexing the current state of open source AI:

> The Gap Map v0.1 details 421 products in depth: 266 software tools and libraries, 85 models, 50 datasets, and 20 hardware projects, produced by 228 organizations. These products are organized into 14 categories across 3 layers of the stack (model components, product / UX, and infrastructure). The remaining 24,400 artifacts constitute the uncategorized long tail of the open source AI ecosystem, and will carry no score until they are researched and cited.

The map itself is interesting to explore, but I'm more excited about the underlying data - released under an MIT license in the [currentai-org/os-ai-map](https://github.com/currentai-org/os-ai-map) GitHub account: 1,184 YAML files plus the notebooks, schemas and other scripts used to help gather them.

Since the files are on GitHub you can use Datasette Lite to explore some of them - here are [16,185 GitHub repos the project is tracking](https://lite.datasette.io/?csv=https://github.com/currentai-org/os-ai-map/blob/main/warehouse/catalog/goodailist/repos.csv#/data/repos?_sort_desc=stars) as a CSV file loaded into Datasette Lite.

Tags: [open-source](https://simonwillison.net/tags/open-source), [ai](https://simonwillison.net/tags/ai), [datasette-lite](https://simonwillison.net/tags/datasette-lite), [generative-ai](https://simonwillison.net/tags/generative-ai), [local-llms](https://simonwillison.net/tags/local-llms), [llms](https://simonwillison.net/tags/llms)
