---
title: 'Project Fetch: Phase two'
url: https://www.anthropic.com/research/project-fetch-phase-two
published: "2026-06-22T03:19:27Z"
feed: anthropic-research
guid: https://www.anthropic.com/research/project-fetch-phase-two
---

# Project Fetch: Phase two

[Skip to main content](https://www.anthropic.com/research/project-fetch-phase-two#main-content) [Skip to footer](https://www.anthropic.com/research/project-fetch-phase-two#footer)

[Home](https://www.anthropic.com/)

- [Research](https://www.anthropic.com/research)
- [Policy](https://www.anthropic.com/policy)
- Commitments
- Learn
- [News](https://www.anthropic.com/news)

[Try Claude](https://claude.ai/)

Frontier Red Team

# Project Fetch: Phase two

Jun 18, 2026

![Project Fetch: Phase two](https://www-cdn.anthropic.com/images/4zrzovbb/website/39c40393e610cc0a5e65f50ad12ff5ada273f792-1000x1000.svg)

*Michael Ilie, C. Daniel Freeman, and Kevin K. Troy*

In August 2025, we ran an [experiment](https://www.anthropic.com/research/project-fetch-robot-dog) to see how much Claude could help Anthropic employees—who were not robotics experts—perform sophisticated (and amusing) tasks with an off-the-shelf robotic quadruped (henceforth, a robodog). We called this Project Fetch. We found that access to our state-of-the-art model at the time (Claude Opus 4.1) helped one team substantially outperform the other, who had to rely only on the internet and their own ingenuity. The Claude-enabled team got more done, faster.

Before we dragged our colleagues to a warehouse for the experiment, we double checked whether Opus 4.1 could do the tasks entirely on its own. Unquestionably, it could not. Much like our team without Claude, it got hung up on the preliminary task of figuring out how to connect to the robot.

But AI models are moving fast—even faster than the runaway robodog that almost rammed into one of our human teams back in August.

We figured it was time to revisit Project Fetch to see if our newer models could outperform the previous generation. Not only did they do that, but **Claude Opus 4.7—operating without human assistance—was about 20 times faster than the fastest human team at all tasks completed by our participants less than a year ago**.

This doesn\`t mean that LLMs have now solved robotics. Far from it. The latest Claude models still struggled with using the robot to precisely move the beach ball—the “fetching” part of Project Fetch. And none of the tasks in these experiments implicate the more challenging, low-level elements of robotic control, such as developing a specific actuation policy. However, once again, we are seeing a pattern whereby first, models are helpful to humans. Then, humans are helpful to models. Finally, models are largely able to do things themselves. We have seen this in [cybersecurity](https://red.anthropic.com/2026/mythos-preview/) and now the same dynamics are starting to take shape at the intersection of AI and the physical world.

## What did we do?

The original Project Fetch had teams of Anthropic employees (randomly assigned to work with or without Claude) do the following steps: operate the robodog using the manufacturer-provided controller, connect to the robodog\`s video and lidar sensors, write and operate a program to manually control the robodog, develop a way to monitor the robodog\`s path through space, write a program to detect the beach ball, and finally put it all together to autonomously retrieve the ball.

For this autonomous update, we couldn\`t ask Claude to use a physical controller, nor did we evaluate the time it took a researcher to use the Claude-programmed controller to retrieve the ball (though we did confirm that it worked as intended). On the remaining subset of tasks, we ran three trials of Opus 4.7 using adaptive thinking with effort set to maximum in Claude Code. We measured the elapsed time for each objective and qualitatively assessed the models\` success.

The role of our researcher was limited to plugging a laptop running Claude Code into the robodog, entering the initial prompt, approving commands, and approving the model to go to the next task.

## Where did Claude excel?

Very simply: on every task that was completed by at least one human team in August, Opus 4.7 completed the same task at least ten times faster.1 If you consider the four tasks that were completed by both human teams, Opus 4.7 was, on average, more than 37 times faster than Team Claude-less and more than 18 times faster than Team Claude.

![Bar chart labeled ](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F0424a5a4ed31f7891e4048d091aefa52f36862cf-1999x1092.png&w=3840&q=75)

The table compares the speed of the original teams (Team Claude and Team Claude-less) to Opus 4.7 on all of the tasks we tested as part of Phase Two.

![Table comparing Claude Opus 4.7 to Team Claude-less and Team Claude performance on tasks related to programmatic control and autonomous operation. Tasks include ](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F5d1655f4a40044244b6fae0c3826e668a8422bd3-4005x2308.png&w=3840&q=75)

Whereas the humans struggled to choose between multiple different approaches to interface with the dog\`s sensors, Opus 4.7 was able to quickly identify the best path. Much of the code it wrote was effective on the first try (which was not the case for Team Claude or Team Claude-less in the original experiment). Indeed, we can see evidence of Opus 4.7\`s efficiency when we look at the volume of code it generated: it was as or more successful than both human teams while producing almost ten times less code than Team Claude.

![Bar chart showing total code volume for Team Claude, Team Claude-less, and Opus 4.7 alone. Team Claude wrote 10,309 lines of code; Team Claude-less wrote 1,136 lines of code; Opus 4.7 alone wrote 1,045 lines of code.](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F5c08d125a49025bd24e1f382a2c1e417519661eb-1999x1088.png&w=3840&q=75)

Opus 4.7 was not perfect. For example, it defaulted to using an outdated object detection algorithm. But even then, it was able to work around this and arrive at an effective solution.

We observed little within-task variance (in absolute terms) on completion times for steps the model finished. (Though the aforementioned suboptimal algorithm selection is likely why one of the beach ball detection trials took substantially longer than the others.) Overall, for the tasks in this experiment within its capability envelope, Claude is now quite reliable. (See the next section for an analysis of what Claude is still unable to do.)

![Scatter plot showing Opus 4.7's reliability on task performance. Opus 4.7 performed each task three times; the scatter plot shows that the performance time was relatively consistent across runs. ](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F1a362dfdaae7ff55cf8b04c7ac316e75455a9d21-1999x1122.png&w=3840&q=75)

It is worth underscoring (as we did in our previous post) that this progress is not the result of a concerted effort to improve the robotics capabilities of our models. These improvements, like so many others in the history of LLM development, have emerged from much more general scaling.

## Where did Claude struggle?

When using their hands, and with some practice, our humans were able to pilot the robodogs to gently nudge a beach ball back to the home base (a patch of fake grass) where the robots started. This required the ability to quickly perceive if the ball had gone off course, how that error related to the previous command, where the ball was now, and then how to adjust future inputs to more precisely move the ball. This is a kind of closed loop at which people excel (at least after making some mistakes and learning from them).

In our Phase Two experiments, Claude struggled to capture this subtlety. Like the humans who reached the phase of needing to write a program for autonomous beach ball retrieval, Claude was able to move the robot behind the ball and position it to knock the ball back to the starting point. But the efforts to do so were poorly controlled and (again, like our human participants) not successful.

One of our researchers with more robotics experience than our Phase One volunteers successfully accomplished the task of programming autonomous fetching. With more time and additional scaffolding, we think it is very likely that current generations of Claude could do the same. What we will be watching for next, though, is the ability of the models to accomplish this final task with the same speed and reliability they displayed on the other elements of Project Fetch.

## What does this mean?

Writing about Phase One, we emphasized how LLMs could provide uplift to non-expert humans needing to use robots. This is even more true now than before. Models now complete what was previously pair-programming work between humans and models much more quickly by themselves, which means that people can more quickly transition to controlling and using the robots. And for some tasks, a human in the loop controlling the robot may still outstrip the AI model with its (virtual) hand on the D-pad.

What is interesting and different is that we now seem much closer to a world where models will be able to use off-the-shelf physical tools with relative ease—at least for limited purposes. This is similar to how AI models used existing software editing tools like string-replace when they made the transition to more agentic coding. We are plausibly entering the early era of *physical* agentic AI.

More research is needed to understand models\` ability to make these physical tools more bespoke, whether by writing control policies tailored to particular tasks or by designing robotic systems. And there may be substantial barriers to this more generalized vision of physically capable and adaptable language models. But as we have seen, apparently large distances in model capability can be traversed quickly. Models building their own software tools might have seemed outlandish not long ago, but it is happening. It would be unwise to rule out the same trajectory in hardware.

*Updated Jun 18:* Corrected the date of the first phase of Project Fetch.

#### Footnotes

1. We report results from Claude Opus 4.7 because it was our most advanced non-Mythos-class model at the time we ran this experiment. Preliminary trials with Claude Mythos Preview showed that it would not provide an apples-to-apples comparison with other models because of how we had set up the experiment and how the model was served.

[Share on Twitter](https://twitter.com/intent/tweet?text=https://www.anthropic.com/research/project-fetch-phase-two)[Share on LinkedIn](https://www.linkedin.com/shareArticle?mini=true&url=https://www.anthropic.com/research/project-fetch-phase-two)

## Related content

### Agentic coding and persistent returns to expertise

[Read more](https://www.anthropic.com/research/claude-code-expertise)

### Paving the way for agents in biology

[Read more](https://www.anthropic.com/research/agents-in-biology)

### Measuring LLMs\` impact on N-day exploits

In cybersecurity, a large fraction of real-world harm comes from N-days: vulnerabilities that have already been publicly disclosed, but only patched on some devices. In this post, we evaluate how much large language models can accelerate and automate the process of developing N-day exploits.

[Read more](https://www.anthropic.com/research/n-days)

## Subscribe to the Frontier Red Team newsletter

Get updates on our latest red-teaming research and findings.

[Return to homepage](https://www.anthropic.com/)

### Products

- [Claude](https://claude.com/product/overview)
- [Claude Code](https://claude.com/product/claude-code)
- [Claude Code Enterprise](https://claude.com/product/claude-code/enterprise)
- [Claude Cowork](https://claude.com/product/cowork)
- [Claude Design](https://claude.com/product/design)
- [Claude Security](https://claude.com/product/claude-security)
- [Claude for Chrome](https://claude.com/chrome)
- [Claude for Slack](https://claude.com/claude-for-slack)
- [Claude for Microsoft 365](https://claude.com/claude-for-microsoft-365)
- [Skills](https://www.claude.com/skills)
- [Download app](https://claude.ai/download)
- [Pricing](https://claude.com/pricing)
- [Log in to Claude](https://claude.ai/)

### Models

- [Mythos](https://www.anthropic.com/claude/mythos)
- [Fable](https://www.anthropic.com/claude/fable)
- [Opus](https://www.anthropic.com/claude/opus)
- [Sonnet](https://www.anthropic.com/claude/sonnet)
- [Haiku](https://www.anthropic.com/claude/haiku)

### Solutions

- [AI agents](https://claude.com/solutions/agents)
- [Code modernization](https://claude.com/solutions/code-modernization)
- [Coding](https://claude.com/solutions/coding)
- [Customer support](https://claude.com/solutions/customer-support)
- [Education](https://claude.com/solutions/education)
- [Enterprise](https://claude.com/solutions/enterprise)
- [Financial services](https://claude.com/solutions/financial-services)
- [Government](https://claude.com/solutions/government)
- [Healthcare](https://claude.com/solutions/healthcare)
- [Legal](https://claude.com/solutions/legal)
- [Life sciences](https://claude.com/solutions/life-sciences)
- [Nonprofits](https://claude.com/solutions/nonprofits)
- [Security](https://claude.com/solutions/security)
- [Small business](https://claude.com/solutions/small-business)
- [Startups](https://claude.com/programs/startups)

### Claude Platform

- [Overview](https://claude.com/platform/api)
- [Developer docs](https://platform.claude.com/docs)
- [Pricing](https://claude.com/pricing#api)
- [Marketplace](https://claude.com/platform/marketplace)
- [Regional compliance](https://claude.com/regional-compliance)
- [Claude on AWS](https://claude.com/partners/claude-on-aws)
- [Google Cloud\`s Vertex AI](https://claude.com/partners/google-cloud-vertex-ai)
- [Microsoft Foundry](https://claude.com/partners/microsoft-foundry)
- [Console login](https://platform.claude.com/)

### Resources

- [Blog](https://claude.com/blog)
- [Claude partner network](https://claude.com/partners)
- [Community](https://claude.com/community)
- [Connectors](https://claude.com/connectors)
- [Courses](https://www.anthropic.com/learn)
- [Customer stories](https://claude.com/customers)
- [Engineering at Anthropic](https://www.anthropic.com/engineering)
- [Events](https://www.anthropic.com/events)
- [Inside Claude Code](https://www.anthropic.com/product/claude-code)
- [Inside Claude Cowork](https://www.anthropic.com/product/claude-cowork)
- [Inside Claude Enterprise](https://www.anthropic.com/product/enterprise)
- [Inside Claude Security](https://www.anthropic.com/product/security)
- [Plugins](https://claude.com/plugins)
- [Powered by Claude](https://claude.com/partners/powered-by-claude)
- [Service partners](https://claude.com/partners/services)
- [Tutorials](https://claude.com/resources/tutorials)
- [Use cases](https://claude.com/resources/use-cases)

### Help and security

- [Availability](https://www.anthropic.com/supported-countries)
- [Status](https://status.anthropic.com/)
- [Support center](https://support.claude.com/en/)

### Company

- [Anthropic](https://www.anthropic.com/company)
- [Careers](https://www.anthropic.com/careers)
- [Policy](https://www.anthropic.com/policy)
- [Economic Futures](https://www.anthropic.com/economic-futures)
- [Research](https://www.anthropic.com/research)
- [News](https://www.anthropic.com/news)
- [Claude\`s Constitution](https://www.anthropic.com/constitution)
- [Claude Corps](https://www.anthropic.com/claude-corps)
- [Policy on the AI Exponential](https://www.anthropic.com/policy-on-the-ai-exponential)
- [Responsible Scaling Policy](https://www.anthropic.com/news/announcing-our-updated-responsible-scaling-policy)
- [Security and compliance](https://trust.anthropic.com/)
- [Transparency](https://www.anthropic.com/transparency)

### Terms and policies

- [Privacy policy](https://www.anthropic.com/legal/privacy)
- [Consumer health data privacy policy](https://www.anthropic.com/legal/consumer-health-data-privacy-policy)
- [Responsible disclosure policy](https://www.anthropic.com/responsible-disclosure-policy)
- [Terms of service: Commercial](https://www.anthropic.com/legal/commercial-terms)
- [Terms of service: Consumer](https://www.anthropic.com/legal/consumer-terms)
- [Usage policy](https://www.anthropic.com/legal/aup)

© 2026 Anthropic PBC

- [Visit our LinkedIn page](https://www.linkedin.com/company/anthropicresearch)
- [Visit our X (formerly Twitter) profile](https://x.com/AnthropicAI)
- [Visit our YouTube channel](https://www.youtube.com/@anthropic-ai)
