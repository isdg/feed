---
title: '[$] Lockless MPSC FIFO queues for io_uring'
url: https://lwn.net/Articles/1081871/
published: "2026-07-15T13:35:43Z"
feed: lwn
guid: https://lwn.net/Articles/1081871/
---

# [$] Lockless MPSC FIFO queues for io_uring

Processes that use [io\_uring](https://man7.org/linux/man-pages/man7/io_uring.7.html) tend to keep a lot of balls in the air; being able to have many operations underway at any given time is part of the point of that API in the first place. The io\_uring subsystem must, as a result, keep track of a lot of tasks that have to be performed at the right time. In current kernels, io\_uring uses a standard kernel linked-list primitive to track those work items. As of the 7.2 kernel release, though, io\_uring will, instead, use a new lockless, multi-producer, single-consumer (MPSC) queue, resulting in some notable performance gains. Lockless algorithms tend to be tricky, but the one used here is relatively approachable and shows how these algorithms can work.
