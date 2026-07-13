---
title: A random variable visualizer
url: https://terrytao.wordpress.com/2026/07/12/a-random-variable-visualizer/
published: "2026-07-13T00:14:31Z"
feed: terrytao
guid: http://terrytao.wordpress.com/?p=17878
---

# A random variable visualizer

With the advent of modern coding agents, many visualization projects that I had proposed in the past, but dropped due to the time and complexity of the coding portion of the task, have now become relatively feasible, in that a reasonable quality prototype (suitable for non-mission-critical tasks such as providing secondary visual aids, where it is not absolutely necessary that the product is 100% bug-free) can now be generated in a matter of hours using such tools. I don’t immediately plan to work on my entire backlog of such projects, but I did spend a few hours this weekend on one such project, namely the proposal from [this 2016 blog post](https://terrytao.wordpress.com/2016/05/13/visualising-random-variables/) to visualize random variables as animated quantities, which can be either viewed numerically or displayed as a scatterplot. After some [back and forth with the coding agent](https://teorth.github.io/tao-web/apps/random-variables-making-of.html), I was able to come up with [a working app](https://teorth.github.io/tao-web/apps/random-variables.html). Here is a screenshot of the app displaying a visualization of [Berkson’s paradox](https://en.wikipedia.org/wiki/Berkson%27s_paradox), which asserts that independent variables can become correlated to each other after applying a conditioning:

[![](https://terrytao.wordpress.com/wp-content/uploads/2026/07/image-3.png?w=1024)](https://terrytao.wordpress.com/wp-content/uploads/2026/07/image-3.png)

The app in fact is a “compiler” for a small, custom programming language (think of a simplified hybrid of Python and Excel) which allows for the introduction of random variables, manipulates them through operations such as arithmetic operations or conditioning, and then plots them as animations or as text. (The screenshot above is static, but when the app is live, it will update at the indicated speed.)

As always, I would be happy to receive feedback on the app, which I hope can be useful as a visual aid to understand basic probabilistic concepts such as independence or conditioning.
