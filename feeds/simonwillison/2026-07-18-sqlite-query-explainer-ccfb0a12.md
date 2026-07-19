---
title: SQLite Query Explainer
url: https://simonwillison.net/2026/Jul/18/sqlite-query-explainer/#atom-everything
published: "2026-07-18T17:19:10Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jul/18/sqlite-query-explainer/#atom-everything
---

# SQLite Query Explainer

**Tool:** [SQLite Query Explainer](https://tools.simonwillison.net/sqlite-query-explainer)

Julia Evan's, in [Learning a few things about running SQLite](https://jvns.ca/blog/2026/07/17/learning-about-running-sqlite/):

> Maybe one day I’ll learn to read a query plan.

Big same.... which inspired me to [have Fable build](https://github.com/simonw/tools/pull/299#issue-4919268017) this interactive explain tool, which runs SQLite in Python in Pyodide in Web Assembly in the browser and adds a layer of explanation to the results of both EXPLAIN and EXPLAIN QUERY PLAN.

Approach with caution, since I don't know enough about SQLite query plans to verify the results myself, but it seems cromulent enough to me.

Tags: [sql](https://simonwillison.net/tags/sql), [sqlite](https://simonwillison.net/tags/sqlite), [tools](https://simonwillison.net/tags/tools), [julia-evans](https://simonwillison.net/tags/julia-evans), [pyodide](https://simonwillison.net/tags/pyodide), [claude-mythos-fable](https://simonwillison.net/tags/claude-mythos-fable)
