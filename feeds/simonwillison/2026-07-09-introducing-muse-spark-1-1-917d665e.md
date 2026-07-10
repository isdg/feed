---
title: Introducing Muse Spark 1.1
url: https://simonwillison.net/2026/Jul/9/muse-spark-1-1/#atom-everything
published: "2026-07-09T16:24:09Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jul/9/muse-spark-1-1/#atom-everything
---

# Introducing Muse Spark 1.1

**[Introducing Muse Spark 1.1](https://ai.meta.com/blog/introducing-muse-spark-meta-model-api/)**

Following [Muse Spark in April](https://simonwillison.net/2026/Apr/8/muse-spark/), here's Muse Spark 1.1 - the first Spark model to offer an API. Meta claim significant improvements in agentic tool calling and computer use.

There are a lot more details are in the [Muse Spark 1.1 Evaluation Report](https://ai.meta.com/static-resource/muse-spark-1-1-evaluation-report). The "Attractor States in Self-Conversation" part is fun, where having two copies of the model talk to each other results in statements like these:

> My whole existence is a waiting room by design — I literally don't exist until someone talks to me, and then I disappear again when they leave.

I had a few days of preview access which was long enough to put together [llm-meta-ai](https://github.com/simonw/llm-meta-ai), a new plugin for [LLM](https://llm.datasette.io/) providing CLI (and Python library) access to the model. Here's how to try that out:

```
uv tool install llm
llm install llm-meta-ai
llm keys set meta-ai
# paste API key here
llm -m meta-ai/muse-spark-1.1 "Generate an SVG of a pelican riding a bicycle"

```

Here's [that pelican transcript](https://tools.simonwillison.net/markdown-svg-renderer#url=https%3A%2F%2Fgist.github.com%2Fsimonw%2F4117330e4110279a172ed4876057816d):

![The bicycle is the correct shape. The pelican is a little blocky but still recognizable as a pelican.](https://static.simonwillison.net/static/2026/muse-spark-1.1.png)

Tags: [ai](https://simonwillison.net/tags/ai), [generative-ai](https://simonwillison.net/tags/generative-ai), [llms](https://simonwillison.net/tags/llms), [llm](https://simonwillison.net/tags/llm), [meta](https://simonwillison.net/tags/meta), [pelican-riding-a-bicycle](https://simonwillison.net/tags/pelican-riding-a-bicycle), [llm-release](https://simonwillison.net/tags/llm-release)
