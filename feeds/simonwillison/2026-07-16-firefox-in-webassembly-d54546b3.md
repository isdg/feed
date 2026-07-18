---
title: Firefox in WebAssembly
url: https://simonwillison.net/2026/Jul/16/firefox-in-webassembly/#atom-everything
published: "2026-07-16T23:34:16Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jul/16/firefox-in-webassembly/#atom-everything
---

# Firefox in WebAssembly

**[Firefox in WebAssembly](https://developer.puter.com/labs/firefox-wasm/)**

This is absurdly cool: Puter compiled Firefox to WebAssembly such that the whole browser runs in another browser.

Here's my blog, running in Firefox, running in WebAssembly, running in Chrome:

![A Chrome window. The tab has the Firefox UI and has loaded my blog. On the right is the Chrome network panel showing that it loaded resources that include a 233MB gecko.wasm and an 18MB chrome-assets.tar.zst](https://static.simonwillison.net/static/2026/firefox-wasm.webp)

They chose Firefox/Gecko because it has strong single-process support. The project used an estimated $25,000 worth of Claude Opus and Fable tokens, but took advantage of a Claude Max subscription plan so cost much less in actual dollars.

The demo funnels all traffic over a WebSocket protocol (using the [Wisp protocol](https://github.com/MercuryWorkshop/wisp-protocol)) through Puter's server - a requirement to get this kind of thing to work because code running in browsers can't open arbitrary network connections.

(That proxying sounds expensive! The team [had to scale the servers up](https://news.ycombinator.com/item?id=48926939#48936563) to handle the traffic during the Hacker News conversation about the project.)

Puter claim this supports end-to-end encryption and that looks to be true - I inspected the WebSocket messages and traffic to my own HTTPS site was encrypted whereas requests and responses to `http://www.example.com/` were in cleartext.

[Here's the repo](https://github.com/HeyPuter/firefox-wasm) for `firefox-wasm`. [theogbob/WebkitWasm](https://github.com/theogbob/WebkitWasm) is a similar project that compiles WebKit to WASM, but that one doesn't currently have an accessible online demo.

Via [Hacker News](https://news.ycombinator.com/item?id=48926939)

Tags: [browsers](https://simonwillison.net/tags/browsers), [firefox](https://simonwillison.net/tags/firefox), [ai](https://simonwillison.net/tags/ai), [webassembly](https://simonwillison.net/tags/webassembly), [generative-ai](https://simonwillison.net/tags/generative-ai), [llms](https://simonwillison.net/tags/llms), [ai-assisted-programming](https://simonwillison.net/tags/ai-assisted-programming), [claude](https://simonwillison.net/tags/claude), [claude-mythos-fable](https://simonwillison.net/tags/claude-mythos-fable)
