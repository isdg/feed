---
title: github-code Web Component
url: https://simonwillison.net/2026/Jul/7/github-code-component/#atom-everything
published: "2026-07-07T16:18:16Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jul/7/github-code-component/#atom-everything
---

# github-code Web Component

**Tool:** [github-code Web Component](https://tools.simonwillison.net/github-code-component)

An experimental Web Component built using GPT-5.5 and [the following prompt](https://gist.github.com/simonw/0e3db21947b5ae7e29e8a4f69a0b0617):

> `let's build a Web Component for embedding code from GitHub`
>
> `<github-code href="https://github.com/simonw/sqlite-ast/blob/437c759129154f05296324a7f82aa1246340dd14/sqlite_ast/parser.py#L9-L18"></github-code>`
>
> `It takes URLs like that, converts them to https://raw.githubusercontent.com/simonw/sqlite-ast/437c759129154f05296324a7f82aa1246340dd14/sqlite_ast/parser.py, then uses fetch() to fetch them and displays the specified range of lines - with line numbers, no syntax highlighting though`
>
> `Show me a preview web browser so I can see your work`

Here's what it looks like embedded on this page:

Tags: [github](https://simonwillison.net/tags/github), [web-components](https://simonwillison.net/tags/web-components), [gpt](https://simonwillison.net/tags/gpt)
