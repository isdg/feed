---
title: 2... 1/2 THEN 3... 1/6 THEN 5 ....1/15 and so on. And So On?
url: https://blog.computationalcomplexity.org/2026/07/2-12-then-3-16-then-5-115-and-so-on-and.html
published: "2026-07-12T22:49:37Z"
feed: complexity
guid: tag:blogger.com,1999:blog-3722233.post-4395052288080959784
---

# 2... 1/2 THEN 3... 1/6 THEN 5 ....1/15 and so on. And So On?

The excellent graphic novel

*Prime Suspects: The Anatomy of Integers and Permutations*

by Andrew Granville and Jennifer Granville,  illustrated by Robert J Lewis,

(I wrote a review of this graphic novel, for SIGACT News, [here](https://www.cs.umd.edu/~gasarch/bookrev/NICK/primesuspects.pdf).)

has an appendix, which is not in graphic-novel form, where they describe some of the math talked about in the graphic novel.

Here is a quote that intrigued me for two reasons

*2 is the smallest prime factor of half of the integers,*

*3 is the smallest prime factor of one-sixth of the integers,*

*5 is the smallest prime factor of one-fifteenth of the integers,*

*and so forth.*

**Intrigue One**: *and so fourth ?* Really? That would indicate that it is easy to know what the next fraction is.  Is it easy? That depends on your definition of *easy.*

**Intrigue Two**: What is the next fraction? What is the fraction asymptotically? I worked  both of these out fairly fast; however, I  don't think  *and so forth* is appropriate.

We derive the 1/15 for 5.  1/5 of all numbers have a factor of 5. Of those, only those that are \\(\\equiv 1,5 \\pmod 6\\) have 5 as the smallest  prime factor.  Hence \\(2/6=1/3\\) of those numbers have 5 as the smallest  prime factor. Hence the fraction is \\(1/5 \\times 1/3 = 1/15\\).

More generally: For all \\(i\\in N\\) let  \\(p\_i\\) be the \\(i\\)th  prime. What fraction of numbers have \\(p\_n\\) as the smallest factor? \\(1/p\_n\\) of all numbers have a factor of \\(p\_n\\). If a number that is divisible by \\(p\_n\\) is also  \\(\\equiv x \\pmod {p\_1p\_2\\cdots p\_{n-1}}\\) where \\(1\\le x\\le p\_1\\cdots p\_n\\) and \\(x\\) is rel prime to \\(p\_1\\cdots p\_n\\), then that number has  \\(p\_n\\) as the smallest factor. Hence the fraction is

\\(\\frac{1}{p\_n} \\times \\frac{\\phi(p\_1\\cdots p\_{n-1})}{p\_1\\cdots p\_n}=\\frac{1}{p\_n} \\times \\frac{(p\_1-1)\\cdots(p\_{n-1}-1)}{p\_1\\cdots p\_{n-1}}= \\frac{1}{p\_n}\\prod\_{i=1}^{n-1} (1-\\frac{1}{p\_i})\\)

where \\(\\phi\\) is the Euler-phi function which, on input \\(x\\), returns the number of naturals in \[1,n\] that are relatively prime to \\(n\\). We have used the following well known facts: (a) if \\(a,b\\) are rel prime then \\(\\phi(ab)=\\phi(a)\\phi(b)\\), and (b) if \\(p\\) is prime then \\(\\phi(p)=p-1\\) (this is obvious).

The expression

\\(\\frac{1}{p\_n} \\times \\frac{(p\_1-1)\\cdots(p\_{n-1}-1)}{p\_1\\cdots p\_{n-1}}\\)

is good for exact calculation. Let's do one!

The fraction of numbers that have 7 as their least prime factor is

\\( \\frac{1}{7}\\times  \\frac{1\\times 2\\times 4}{2\\times 3\\times 5}= \\frac{4}{105} \\)

From the first three terms  \\( \\frac{1}{2}, \\frac{1}{3}, \\frac{1}{15} \\) I do not think that *and so fourth* would make anyone think the next term was \\(\\frac{4}{105}\\).

But what is the fraction asymptotically?

ChatGPT tells me (and I believe it) that

\\( \\prod\_{i=1}^{n} (1-\\frac{1}{p\_i})   \\sim \\frac{e^{-\\gamma}}{\\ln p\_n} \\)

where \\(\\gamma\\) is the Euler-Mascheroni constant, roughly 0.5772 (see [here](https://en.wikipedia.org/wiki/Euler%27s_constant) for the Wikipedia entry on that constant).

Hence we get that the fraction of numbers that have \\(p\_n\\) as their smallest prime factor is

\\(\\frac{1}{p\_n} \\prod\_{i=1}^{n-1} (1-\\frac{1}{p\_i})   \\sim\\frac{1}{p\_n} \\frac{e^{-\\gamma}}{\\ln p\_{n-1}} \\)

Wolfram Alpha tells me (and I believe it) that

\\(e^{-0.5772}\\sim 0.56146\\).

Hence we give our final approx for the fraction of numbers that have \\(p\_n\\) as their least prime factor as

\\(\\frac{0.56146}{p\_n\\ln(p\_{n-1})}\\)

I don't think this qualifies as *and so forth.*
