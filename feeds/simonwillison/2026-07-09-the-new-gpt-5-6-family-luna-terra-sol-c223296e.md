---
title: 'The new GPT-5.6 family: Luna, Terra, Sol'
url: https://simonwillison.net/2026/Jul/9/gpt-5-6/#atom-everything
published: "2026-07-09T19:46:38Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jul/9/gpt-5-6/#atom-everything
---

# The new GPT-5.6 family: Luna, Terra, Sol

OpenAI's latest flagship model [hit general availability this morning](https://openai.com/index/gpt-5-6/), and comes in three sizes: Luna, Terra, and Sol (from smallest to largest).

The new models are priced per 1M input/output tokens as Luna $1/$6, Terra $2.50/$15, Sol $5/$30. For comparison, the Claude Opus series are $5/$25 and the Claude Fable 5 is $10/$50, but price-per-million tokens doesn't tell us much now that the number of reasoning tokens can differ so much between models for the same task.

All three models have a February 16th 2026 knowledge cutoff, a million token context window, and 128,000 maximum output tokens.

OpenAI's biggest benchmark claim concerns long-running agentic performance, with one benchmark showing all three models outperforming Claude Fable 5:

> We trained GPT-5.6 to get more useful work from every token. On [Agents’ Last Exam](https://agents-last-exam.org/), an evaluation of long-running professional workflows across 55 fields, GPT-5.6 Sol sets a new high of 53.6, eclipsing Claude Fable 5 (adaptive reasoning) by 13.1 points. Even at medium reasoning, it beats Fable 5 by 11.4 points at roughly one-quarter the estimated cost. That efficiency extends to smaller models, which are essential to making intelligence more abundant and affordable: GPT-5.6 Terra and GPT-5.6 Luna outperform Fable 5 at around one-sixteenth the cost.

Amusingly, one self-reported benchmark that Fable 5 crushed the GPT-5.6 family on was SWE-Bench Pro, where Fable 5 got 80% compared to GPT-5.6 Sol getting 64.6%. This may help explain why OpenAI chose to publish [this article yesterday](https://openai.com/index/separating-signal-from-noise-coding-evaluations/) specifically calling out SWE-Bench Pro for problems they found while auditing that benchmark:

> In light of these results, we estimate that ~30% of SWE-bench Pro tasks are broken, and advise that model developers carefully examine results

I've had some early access to GPT-5.6 Sol - it's definitely very competent, though so far it hasn't struck me as better than Fable at the kind of complex coding tasks I've been using with Anthropic's model.

As usual, the [model guidance for using GPT-5.6](https://developers.openai.com/api/docs/guides/latest-model?model=gpt-5.6) has the most interesting details. There are a bunch of new API features that I need to explore (and probably add support for in [LLM](https://llm.datasette.io/)), including:

- [Programmatic Tool Calling](https://developers.openai.com/api/docs/guides/tools-programmatic-tool-calling) allows the models to "compose and run JavaScript that orchestrates tool calls" - which sounds to me like it could help bridge the gap between MCPs and full terminal sessions that can compose CLI utilities in useful ways. Also reminiscent of the [dynamic filtering](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool#dynamic-filtering) mechanism Anthropic added to their web search tool, which allows code execution against web results as part of a single model turn.
- [Multi-agent](https://developers.openai.com/api/docs/guides/tools-multi-agent) lets the model "spin up subagents for parallel, focused work" - the sub-agent pattern now baked into the core API.
- [Prompt cache breakpoints](https://developers.openai.com/api/docs/guides/prompt-caching#prompt-cache-breakpoints) brings the Claude model of prompt caching to OpenAI, letting you be explicit about where the cache breakpoints are rather than relying on the API to detect them automatically. Personally I much prefer automatic detection (still supported by OpenAI), but presumably there are optimization cost savings to be had here if you put the work in.
- You can now set [detail: original](https://developers.openai.com/api/docs/guides/images-vision#choose-an-image-detail-level) on image requests to avoid resizing the image at all before it is processed.

Here's [a full page with 18 different pelicans](https://static.simonwillison.net/static/2026/gpt-5.6-pelicans.html) \- for reasoning efforts none, low, medium, high, xhigh, and max across the three different models. It also lists their token and calculated costs - the least expensive was gpt-5.6-luna at effort none for 0.71 cents, the most expensive was gpt-5.6-sol at max reasoning level for 48.55 cents.

![A grid of nine pelicans riding bicycles, of varying quality](https://static.simonwillison.net/static/2026/gpt-5.6-pelicans.webp)

In further pelican news, if you jump to 17:50 in [their livestream from this morning](https://www.youtube.com/live/Wq45rvPGNHs?t=1070s) you'll see OpenAI's own demo of 3D pelicans riding a tricycle, a bicycle, a pony, and another pelican!

![Frame from a livestream showing a 3D model of a pelican riding another pelican](https://static.simonwillison.net/static/2026/pelican-riding-a-pelican.jpg)

Tags: [ai](https://simonwillison.net/tags/ai), [openai](https://simonwillison.net/tags/openai), [generative-ai](https://simonwillison.net/tags/generative-ai), [llms](https://simonwillison.net/tags/llms), [llm-tool-use](https://simonwillison.net/tags/llm-tool-use), [llm-pricing](https://simonwillison.net/tags/llm-pricing), [pelican-riding-a-bicycle](https://simonwillison.net/tags/pelican-riding-a-bicycle), [llm-release](https://simonwillison.net/tags/llm-release), [gpt-5](https://simonwillison.net/tags/gpt-5)
