---
title: Curse of the CEMBI / Let Maintainers Be Maintainers
url: https://graydon2.dreamwidth.org/306832.html
published: "2023-03-28T04:45:51Z"
feed: graydon
guid: tag:dreamwidth.org,2011-01-04:684470:306832
---

# Curse of the CEMBI / Let Maintainers Be Maintainers

A short note about corporate free / open source software (FOSS),
and corporate-employed maintainers. Or specifically
"corporate-employed maintainers .. with bad incentives". I tried
to come up with a pithy name, but that's the best I could do:
CEMBIs.

I've worked professionally in FOSS for a long time. I've seen a
lot of corporate approaches to FOSS participation over that time,
and seen the FOSS community develop a fairly nuanced understanding
of what sorts of corporate strategies are welcome or unwelcome,
healthy or harmful to a project. We have articulated a lot of
thoughts about (say) open core and dual licensing business models,
or (say) which forms of corporate "embrace" represent a step on an
"extend/extinguish" path, or (say) which forms of telemetry are
appropriate and which put users at risk of surveillance.

What I haven't seen a lot of discussion of, and wish I did see, is
the structure and content of relationships that exist between
corporate-employed FOSS maintainers and their employers. And I
think this matters because the people doing corporate FOSS aren't
soul-less automata executing corporate strategy. They are people
with their own motivations, incentives, a certain amount of
autonomy, but (most relevant to my concerns) a set of
performance-evaluation criteria they have to satisfy to remain
employed and/or get promoted.

Companies incentivize lots of things, but I worry (and in many
cases I either know first hand or have heard second hand) that
companies often have an incentive structure that rewards
_novelty_, especially in the form of _features_ if not entire
_products_. There's a good reason for this when the company is
"making products to sell": this year's new-and-improved is always
sellable over last year's tired old model. But it's also just a
reflection of the "growth orientation" or capitalism in general:
whatever activity the company accomplished last year, a common
measure of health and vitality isn't consistent execution but
_growth of the business_. Failure to grow is treated as stagnation
which is treated as equivalent to death. This orientation can be
embedded so deep in a company's DNA that it's the incentive
structure given to _everyone_ who works there, regardless of
whether they're making products, auditing the accounts, or
_maintaining corporate infrastructure_.

And to a large measure, that's what FOSS is. Not always, but
usually:
[infrastructure](https://en.wikipedia.org/wiki/Infrastructure).
It's stuff that's supposed to work the same way from one day to
the next. Stuff that's not supposed to be noticed because it just
works. Reliably, efficiently, silently. Stuff that has a massive
installed base of users relying on it, massive social and
institutional inertia, and thereby massive (and sensible) built-in
resistance to novelty. You don't actually want novelty in the
electrical grid, the drinking water system, sewers, roads,
bridges, rail lines, telecoms .. you want this stuff to be
_absolutely rock solid_ and not novel in the least. That's what
maybe 90 or 95% of FOSS is like, certainly the stuff that _needs_
reliable corporate maintainers.

For corporate-employed FOSS maintainers working at a firm with
these "growth and novelty" incentives -- CEMBIs -- this leads to a
real quandry. They're in a position where their job performance is
very likely to be evaluated in terms of visible growth and novelty
(it might be dressed up in more abstract terms like "real-world
impact" or "visibility" but it still means the same thing) even
though that is exactly the wrong thing for the health of the
project they're maintaining. The incentives are bad. If they do
the _best_ thing for the project _as infrastructure_ \-\- triage
and fix bugs from the backlog, optimize performance, increase
security and reliability, pay down tech debt, simplify and
automate ongoing maintenance -- the bias of their organization is
likely to view them as "doing nothing", or at best doing
"low-value work" that only counts as "reducing fixed costs", not
leading the way towards new growth. To be seen in a positive light
by their employer, the CEMBI winds uphaving to do essentially
_anti-maintenance work_: make the program "do something new and
exciting", that it didn't do yesterday. Ignore maintenance of
"what is", focus on "what's next".

Seeing this over and over in FOSS makes me a bit sad. I guess I've
maybe been watching too many re-runs of The West Wing lately, but
while I've been thinking about this subject, I keep thinking of
the "
[Let Bartlet Be Bartlet](https://en.wikipedia.org/wiki/Let_Bartlet_Be_Bartlet)"
episode near the end of the first season, where the characters
come around to the idea that maybe it'd be best to just do what
they were elected to do. I keep thinking it'd be nice if employers
could incentivize their employees to do what they were hired to
do. To " **Let Maintainers Be Maintainers**". Maintaining FOSS
isn't low-value work. It's _essential_ work, stuff that you ignore
at your medium and long-term peril. As a new homeowner I will make
more salient analogies: it's testing the wiring for faults,
testing the walls for mould, replacing the roof before it leaks.
It's work that _has to be done_ in order for FOSS to keep
functioning over the long term. Software actually does "break down
and wear out": requirements change; upstream and downstream
platforms and libraries change incompatibly; patterns of usage
change; hardware changes; new subsystems become performance
bottlenecks; new bugs are discovered, some of which will be high
severity security vulnerabilities, others will merely grow in
importance as they're encountered more often. Software maintenance
is _real and important work_, and if a company hires a maintainer
of a project "in order to support the project", it's what that
company should be incentivizing and evaluating those maintainers
in terms of.

![comment count unavailable](https://www.dreamwidth.org/tools/commentcount?user=graydon2&ditemid=306832)
comments
