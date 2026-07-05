---
title: 'Better Models: Worse Tools'
url: https://simonwillison.net/2026/Jul/4/better-models-worse-tools/#atom-everything
published: "2026-07-04T22:53:52Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jul/4/better-models-worse-tools/#atom-everything
---

# Better Models: Worse Tools

**[Better Models: Worse Tools](https://lucumr.pocoo.org/2026/7/4/better-models-worse-tools/)**

Armin reports on a weird problem he ran into while hacking on Pi:

> The short version is that newer Claude models sometimes call Pi’s edit tool with extra, invented fields in the nested `edits[]` array. And not Haiku or some small model: Opus 4.8. The edit itself is usually correct but the arguments do not match the schema as the model invents made-up keys and Pi thus rejects the tool call and asks to try again.
>
> That alone is not too surprising as models emit malformed tool calls sometimes. Particularly small ones. What surprised me is that this is getting worse with newer Anthropic models as both Opus 4.8 and Sonnet 5 show it but none of the older models. In other words, the SOTA models of the family are worse at this specific tool schema than their older siblings.

Armin theorizes that this is because more recent Anthropic models have been specifically trained (presumably via Reinforcement Learning) to better use the edit tools that are baked into Claude Code. This has the unfortunate effect that other coding harnesses, such as Pi, may find that their own custom edit tools are more likely to be used incorrectly.

Claude's edit tool [uses search and replace](https://platform.claude.com/docs/en/agents-and-tools/tool-use/text-editor-tool#str-replace). OpenAI's Codex [uses an apply\_patch mechanism instead](https://developers.openai.com/api/docs/guides/tools-apply-patch), and OpenAI have talked in the past about how their models are trained to use that tool effectively.

Does this mean third-party coding harnesses like Pi should implement multiple edit tools just so they can use the one with the best performance for the underlying model the user has selected?

Tags: [armin-ronacher](https://simonwillison.net/tags/armin-ronacher), [ai](https://simonwillison.net/tags/ai), [openai](https://simonwillison.net/tags/openai), [generative-ai](https://simonwillison.net/tags/generative-ai), [llms](https://simonwillison.net/tags/llms), [anthropic](https://simonwillison.net/tags/anthropic), [llm-tool-use](https://simonwillison.net/tags/llm-tool-use), [coding-agents](https://simonwillison.net/tags/coding-agents), [pi](https://simonwillison.net/tags/pi)
