---
title: Using DSPy to evaluate and improve Datasette Agent's SQL system prompts
url: https://simonwillison.net/2026/Jul/2/dspy-datasette-agent-prompts/#atom-everything
published: "2026-07-02T18:25:00Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jul/2/dspy-datasette-agent-prompts/#atom-everything
---

# Using DSPy to evaluate and improve Datasette Agent's SQL system prompts

**Research:** [Using DSPy to evaluate and improve Datasette Agent's SQL system prompts](https://github.com/simonw/research/tree/main/dspy-datasette-agent-prompts#readme)

One of this morning's AIE keynotes covered [dspy](https://github.com/stanfordnlp/dspy), which reminded me I've been meaning to see if it could help me improve the system prompt used by [Datasette Agent](https://agent.datasette.io) \- so I fired off an asynchronous research task in Claude Code for web using Claude Fable 5:

> `Pip install the latest Datasette alpha and datasette-agent and dspy - then figure out how to use dspy to evaluate and improve the main system prompts used by Datasette Agent for the feature where it can execute read only SQL queries to answer user questions about data.`

Fable chose to test using GPT 4.1 mini and nano, and identified several promising looking directions for improvements. I particularly like this one:

> The schema listing gives only table names; the "don't call describe\_table if you already have the information" advice caused column-name guessing (page\_count, o.order\_id, first\_name) and error-retry loops in baseline traces. Either include column names in the prompt's schema listing or soften that advice.

Tags: [ai](https://simonwillison.net/tags/ai), [datasette](https://simonwillison.net/tags/datasette), [generative-ai](https://simonwillison.net/tags/generative-ai), [llms](https://simonwillison.net/tags/llms), [evals](https://simonwillison.net/tags/evals), [dspy](https://simonwillison.net/tags/dspy), [datasette-agent](https://simonwillison.net/tags/datasette-agent), [claude-mythos-fable](https://simonwillison.net/tags/claude-mythos-fable)
