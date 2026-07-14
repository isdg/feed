---
title: Using uvx in GitHub Actions in a cache-friendly way
url: https://simonwillison.net/2026/Jul/14/uvx-github-actions-cache/#atom-everything
published: "2026-07-14T00:56:20Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jul/14/uvx-github-actions-cache/#atom-everything
---

# Using uvx in GitHub Actions in a cache-friendly way

**TIL:** [Using uvx in GitHub Actions in a cache-friendly way](https://til.simonwillison.net/github-actions/uvx-github-actions-cache)

I finally found a cache-friendly recipe for using `uvx tool-name` in GitHub Actions workflows that I like.

The trick is setting a `UV_EXCLUDE_NEWER: "2026-07-12"` environment variable at the start of the workflow and then using that as part of the GitHub Actions cache key. This means any `uvx tool-name` commands will resolve to the most recent version as-of that date, and you can bust the cache and upgrade the tools by bumping the date in the future.

My goal here is to use Python tools in GitHub Actions without every run of the workflow hitting PyPI to download a fresh copy of the tool and its dependencies.

**Update**: Here's an existing [issue](https://github.com/astral-sh/setup-uv/issues/745) against the `astral-sh/setup-uv` repository requesting that they switch the default to cache rather than purge wheels from PyPI.

Tags: [packaging](https://simonwillison.net/tags/packaging), [pypi](https://simonwillison.net/tags/pypi), [python](https://simonwillison.net/tags/python), [github-actions](https://simonwillison.net/tags/github-actions), [uv](https://simonwillison.net/tags/uv)
