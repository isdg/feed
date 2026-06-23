---
title: The New Result on Off-diagonal Ramsey Numbers
url: https://blog.computationalcomplexity.org/2026/06/the-new-result-on-off-diagonal-ramsey.html
published: "2026-06-22T11:55:05Z"
feed: complexity
guid: tag:blogger.com,1999:blog-3722233.post-1305115097196649740
---

# The New Result on Off-diagonal Ramsey Numbers

(All references in this blog post can be found in the main article the post is about which is [here](https://arxiv.org/abs/2605.28793).)

Recall that \\(R(a,b)\\) is the least \\(n\\) so that, for all 2-colorings of the edges of \\(K\_n\\), there is either a RED \\(a\\)-clique or a BLUE \\(b\\)-clique.

\\(R(k,k)\\) has been well studied and is often called \\(R(k)\\).

However, today we are concerned with \\(R(a,k)\\) \\(a\\) is fixed and \\(k\\) goes to infinity.

1) In 1995 Jeong Han Kim showed \\(R(3,k)\\) is asy \\(\\Theta(\\frac{k^2}{\\log k})\\). At the workshop In Ramsey Theory: Yesterday, Today, and Tomorrow, Edited by Alexander Soifer, 2011, i Joel Spencer give a great talk titled

80 years of \\(R(3,k)\\).

The title implies that the problem was open for 80 years, but 40 years is a better estimate.

The general sense I got from both Joel and the audience is that \\(R(4,k)\\) is a much harder problem.

2) In 2023 there was substantial progress on \\(R(4,k)\\). Sam Mattheus and Jacques Verstraete showed

\\(R(4,k) = \\Omega(\\frac{k^3}{\\log^4 k})\\)

Combined with prior results this yields

\\( c\_1 \\frac{k^3}{\\log^4 k} \\le R(4,k) \\le c\_2 \\frac{k^3}{\\log^2 k} \\)

The prior lower bound had been \\(\\Omega(\\frac{k^{2.5}}{\\log^2 t})\\) so this was a big improvement.

3) The following was known for \\(R(s,k)\\) for fixed \\(s\\) and asy \\(k\\):

\\( k^{(s+1)/2 + o(1)} \\le O(k^{s-1}) \\)

There were polylog improvements to this which we omit.

Surely improving the lower bound to \\(\\Omega(k^{s-1+o(1)})\\) would be hard.

4) Domagoj Bradac showed the following (posted on May 27, 2026, and pointed to at the beginning of this post)

\\(R(s,k) \\ge \\Omega(\\frac{k^{s-1}}{(\\log k)^{2s-4}}).\\)

Combining this with the known best upper bounds yields (ignoring constants)

\\( \\frac{k^{s-1}}{(\\log k)^{2s-4}} \\le R(k,s) \\le (1+o(1))\\frac{k^{s-1}}{(\\log k)^{s-2}} \\)

So the bounds are now only a polylog apart!

Random Notes.

1) I am not surprised about the results.

2) I am very surprised that the results were obtained.

3a) In 2011 most people thought that \\(R(4,k)\\) would be hard.

3b) After the progress on \\(R(4,k)\\) I still thought that \\(R(5,k)\\) would be hard, though I don't know what others thought.

4) Why such fast progress?

4a) AI? Some was used but not that much. See comment on page 4 of the paper.

4b) More people working on the problem?

4c) More advanced tools developed over time? Surely yes.

5) What other problem that seems like its solution is long in coming will be solved much faster than we think?
