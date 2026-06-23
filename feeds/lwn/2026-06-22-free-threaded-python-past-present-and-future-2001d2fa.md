---
title: '[$] Free-threaded Python: past, present, and future'
url: https://lwn.net/Articles/1078367/
published: "2026-06-22T15:26:54Z"
feed: lwn
guid: https://lwn.net/Articles/1078367/
---

# [$] Free-threaded Python: past, present, and future

Probably the biggest change for Python over the last five years or so is the advent of the "free-threaded" version of the language, which removes the global interpreter lock (GIL) and allows multiple threads to run in parallel in the interpreter. At [PyCon
US 2026](https://us.pycon.org/2026/), held in Long Beach, California in mid-May, longtime CPython core developer (and current steering council member) Thomas Wouters gave a talk about the feature. He looked at the motivation behind the GIL-removal efforts, some history, the current status of the free-threaded interpreter, and provided a prediction on where it all leads.
