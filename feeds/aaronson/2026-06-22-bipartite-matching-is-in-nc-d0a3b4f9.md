---
title: Bipartite matching is in NC!
url: https://scottaaronson.blog/?p=9851
published: "2026-06-22T17:27:42Z"
feed: aaronson
guid: https://scottaaronson.blog/?p=9851
---

# Bipartite matching is in NC!

Since I’m a good mood today—at a beautiful science camp with my kids, high in the mountains near Big Bear Lake in California—I thought I’d blog about something positive. Last week, five authors posted a [major paper](https://eccc.weizmann.ac.il/report/2026/100/) to the Electronic Colloquium on Computational Complexity, which shows (or anyway, credibly claims to show) that the [Bipartite Matching](https://discrete.openmathbooks.org/dmoi3/sec_matchings.html) problem is in the complexity class [NC](https://en.wikipedia.org/wiki/NC_(complexity)). Assuming this stands, it resolves a central problem in parallel algorithms and derandomization that’s been open since the 1980s.

In Bipartite Matching, you’re given a list of n men and n women, you’re told who’s willing to date whom, and your goal is to

1. decide whether it’s possible to pair everyone off with a willing partner, and
2. if they are, actually pair them off.

One of the great early discoveries of combinatorial algorithms, taught in every introductory algorithms course, is that this problem is solvable in time polynomial in n, even though the naïve, brute-force approach would require examining n! possibilities.

(Note that in the *bipartite* version, we assume that the men and women are all straight. If the men and women can be LGBT, we get the problem of matching in *general* graphs, which again turns out to be solvable in polynomial time, but now the algorithm is much more sophisticated, and was a major discovery of Edmonds in the 1960s.)

Anyway, the question is whether we can do even better than polynomial time: in particular, can we solve the problem in *logarithmic* time, given polynomially many parallel processors?

Back in the 1980s, Ketan Mulmuley, my former PhD adviser Umesh Vazirani, and Umesh’s brother Vijay Vazirani managed to [show that the answer is yes](https://people.eecs.berkeley.edu/~vazirani/pubs/matching.pdf), but only if the parallel processors additionally get access to random bits, and only need to succeed with high probability.

The new achievement is to derandomize Mulmuley-Vazirani-Vazirani, and show that problems 1 and 2 above are both solvable in *deterministic* logarithmic time with parallel processing (in other words, in the complexity class NC).

No, I don’t understand how it works yet. If anyone does, feel free to explain in the comments! Or ask your favorite AI to generate a summary. If I run out of options, at some point I might actually try *reading the paper*.

---

**One other announcement:** Tomorrow is the day of primary elections in NYC! Virtually *all* of my smartest friends who work on AI governance and safety are extremely excited about the Congressional campaign of [Alex Bores](https://www.alexbores.nyc/)—indeed, it would be little exaggeration to say that they consider him the last best hope of humankind. Bores has been a national leader on trying to regulate AI, to the extent that Marc Andreessen’s “Leading the Future” anti-AI-regulation PAC has spent millions of dollars trying to sink his candidacy. Outside of AI, Bores seems like a sane, conventional Democrat, i.e. the kind I like, and much more moderate than his base on Israel (note that his main opponent is also such). Without commenting on Bores’ views on every possible issue, I’ll simply say: if you live in [New York’s 12th Congressional District](https://en.wikipedia.org/wiki/New_York%27s_12th_congressional_district) (comprising a huge chunk of central Manhattan), and you care about AI safety, please consider a vote for Bores while there’s still time.
