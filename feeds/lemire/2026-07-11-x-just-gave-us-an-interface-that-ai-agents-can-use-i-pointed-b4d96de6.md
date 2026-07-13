---
title: X just gave us an interface that AI agents can use. I pointed it at my own posts.
url: https://lemire.me/blog/2026/07/11/x-just-gave-us-an-interface-that-ai-agents-can-use-i-pointed-it-at-my-own-posts/
published: "2026-07-11T19:53:17Z"
feed: lemire
guid: https://lemire.me/blog/?p=22714
---

# X just gave us an interface that AI agents can use. I pointed it at my own posts.

![](https://lemire.me/blog/wp-content/uploads/2026/07/image-5-150x150.jpg)

I have been on X for a long time. Like most people who post regularly, I have a gut feeling for what might interest people. I post in the morning. Longer posts seem to do better.

But gut feelings are not measurements. And until recently, digging into your own posting data meant either clicking around the web UI or writing custom scripts. Neither is particularly friendly when you want to ask *ad hoc* questions with an AI assistant.

X recently launched hosted [MCP](https://modelcontextprotocol.io/) servers: official endpoints that AI tools can connect to. MCP is a protocol for plugging tools into language models: the model can search posts, manage bookmarks, fetch trends, and so on. In practice, I connected an AI coding agent to the X MCP server and simply started asking questions about my account.

I spent a session exploring about two months of my own activity. Here is what I found interesting.

Over roughly sixty days (mid-May through mid-July 2026), I published on the order of 435 posts that were not pure retweets of other people—mostly a mix of original posts, replies, and a few X Articles. The agent pulled them through the MCP tools, kept the public metrics (likes, views, reposts), and ran simple analyses.

I asked for every post to be binned by local hour of day (America/Toronto, Eastern time), and for each hour: how many posts, and the min / median / max view count.

My posting is heavily skewed toward the morning:

Local hourPostsMedian views08:00–08:594545409:00–09:59581,06710:00–10:593019411:00–11:5942284

The 9 a.m. hour is both my busiest and, among busy hours, my strongest by median views. The overall median across all hours was only about 188 views, so most of what I write is quiet. The distribution is heavy-tailed: a few posts get tens or hundreds of thousands of impressions; the rest are background noise.

I then binned posts by character length in steps of 25 characters (using the text as returned by the API, including short `t.co` URLs).

The bulk of my writing is short, often a reply of a few dozen characters:

CharactersPostsMedian likesMax likes0–2544119525–508716050–756909875–10051161100–12533132…175–20012558200–225174456275–300194385300–3254846.5470

Under about 175 characters, the median stays at zero or one like. Around full-length posts (roughly the old 280-character regime and a bit beyond), engagement jumps. The 300–325 character band is where a large fraction of my “serious” posts live, and the median likes there are an order of magnitude higher than for short replies.

I also asked the AI to identify the posts that had the most likes, the following types of posts were liked:

- AI vs. “experts” claiming models are nowhere near human intelligence
- Go adding SIMD-style data-parallelism to the standard library
- SIMD-accelerated data processing talks and library notes (JSON, string→integer maps, vulnerability-report fatigue)
- Nvidia hardware, university AI-cheating, C++ contracts

The interesting part is the workflow. I did not export a CSV by hand and open Excel. I asked an agent, connected to X’s MCP server. If AI agents can do this for one account’s metrics, they can do it for bug trackers, logs, paper drafts, and codebases.
