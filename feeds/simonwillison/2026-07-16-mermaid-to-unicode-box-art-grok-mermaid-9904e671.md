---
title: Mermaid to Unicode box art (grok-mermaid)
url: https://simonwillison.net/2026/Jul/16/grok-mermaid/#atom-everything
published: "2026-07-16T00:33:18Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jul/16/grok-mermaid/#atom-everything
---

# Mermaid to Unicode box art (grok-mermaid)

**Tool:** [Mermaid to Unicode box art (grok-mermaid)](https://tools.simonwillison.net/grok-mermaid)

While [exploring the codebase](https://simonwillison.net/2026/Jul/15/grok-build/) for the newly open-sourced Grok CLI coding agent I came across [xai-grok-markdown/src/mermaid.rs](https://github.com/xai-org/grok-build/blob/b189869b7755d2b482969acf6c92da3ecfeffd36/crates/codegen/xai-grok-markdown/src/mermaid.rs), a "self-contained terminal renderer for Mermaid diagrams" written in Rust.

I figured it would be fun to try that out in a browser via WebAssembly. Here's [the prompt](https://github.com/simonw/tools/pull/293#issue-4897479396) I ran in Claude Code for web (Fable 5), and this is what the resulting tool looks like:

![Screenshot of a Mermaid diagram editor showing source code and rendered flowchart. The code reads: graph TD Start[Request received] --> Auth{Authenticated?} Auth -->|yes| Rate{Rate limit OK?} Auth -->|no| R401[401 Unauthorized] Rate -->|yes| H(Handle request) Rate -->|no| R429[429 Too Many Requests] H -.-> Log[Audit log] H ==> Resp[200 OK]. Below the code are controls labeled Max width: Fit output panel, Copy as text, and Copy link to this diagram. The rendered flowchart on a dark background flows top-down: Request received leads to Authenticated?, which branches yes to Rate limit OK? and no to 401 Unauthorized. Rate limit OK? branches yes to Handle request and no to 429 Too Many Requests. Handle request connects with a dotted arrow to Audit log and a thick arrow to 200 OK.](https://static.simonwillison.net/static/2026/grok-mermaid-wasm.png)

Tags: [tools](https://simonwillison.net/tags/tools), [rust](https://simonwillison.net/tags/rust), [webassembly](https://simonwillison.net/tags/webassembly), [mermaid](https://simonwillison.net/tags/mermaid), [grok](https://simonwillison.net/tags/grok), [xai](https://simonwillison.net/tags/xai)
