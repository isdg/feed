---
title: An off switch for dual use knowledge in AI models
url: https://www.anthropic.com/research/off-switch-dual-use
published: "2026-07-09T03:45:40Z"
feed: anthropic-research
guid: https://www.anthropic.com/research/off-switch-dual-use
---

# An off switch for dual use knowledge in AI models

[Skip to main content](https://www.anthropic.com/research/off-switch-dual-use#main-content) [Skip to footer](https://www.anthropic.com/research/off-switch-dual-use#footer)

[Home](https://www.anthropic.com/)

- [Research](https://www.anthropic.com/research)
- [Policy](https://www.anthropic.com/policy)
- Commitments
- Learn
- [News](https://www.anthropic.com/news)

[Try Claude](https://claude.ai/)

Alignment

# An off switch for dual use knowledge in AI models

Jul 8, 2026

![An off switch for dual use knowledge in AI models](https://www-cdn.anthropic.com/images/4zrzovbb/website/ba5df68fde631a953fcaca4ec56cd20d285bdd5d-1000x1000.svg)

*This post describes research conducted by [AE Studio](https://ae.studio/alignment) in collaboration with Anthropic.*

A frontier AI model is, among other things, a large store of knowledge. Some of that knowledge is *dual use*, meaning it can be used for good or for bad. For example, knowledge of cybersecurity can help patch critical security vulnerabilities, or it can be used to exploit them. Knowledge of virology can help a researcher create a vaccine, but it can also help a malicious actor design a deadly pathogen. Ideally, we would be able to balance three separate goals: first, limiting access to dual use capabilities in as surgical a way as possible; second, allowing trusted users to access those same capabilities for beneficial purposes; and third, do all this without affecting the model\`s performance on any other task.

Current safeguards are imperfect. We train models to refuse harmful requests and use classifiers to screen inputs and outputs for dangerous content. These layers of protection guard against dangerous outputs—but they don\`t change the knowledge stored in the underlying model. Despite our safeguards, a sufficiently determined attacker may still try to [jailbreak](https://www.anthropic.com/news/fable-safeguards-jailbreak-framework) the model, working past its defenses to access the dual use knowledge.

A more robust protection against misuse would be to control what the model knows. We\`ve explored this before: in earlier work, we [filtered information about chemical, biological, radiological, and nuclear weapons out of pretraining data](https://alignment.anthropic.com/2025/pretraining-data-filtering/), and later showed that [dual-use knowledge can be confined to a removable slice of a model\`s weights](https://alignment.anthropic.com/2025/selective-gradient-masking/). But filtering is a blunt instrument. It produces one model with one fixed set of capabilities. Using filtering, if you want a model version that *can* discuss advanced virology—for deployment in a vetted biosecurity lab, say—and another version that can\`t, you have to train two separate models. Especially in the case of frontier models (which are large and very expensive to train), the cost to the developer would be prohibitive.

In [new research](https://alignment.anthropic.com/2026/modular-pretraining/) carried out with collaborators at AE Studio, we explore a new method that could enable the benefits of training many separately-filtered models, but at the cost of training only one model. We call it GRAM, for Gradient-Routed Auxiliary Modules. Note that the results of the experiments presented here are preliminary—GRAM has not been applied to any of the production models at Anthropic, and we\`re not sure it ever will be.

## How GRAM works

The idea behind GRAM is to give a model dedicated, removable compartments for each category of dual-use knowledge, and to update only those compartments when learning from dual-use data.

Concretely, GRAM adds extra neurons to every layer of a standard Transformer (the neural network architecture on which large language models are based). These neurons are divided into groups (or “modules”), one per dual-use category. During training, when the model encounters general-purpose text, it learns in the usual way. But when it encounters text from a dual-use category—virology, for instance—the rules change: the model can *use* its general knowledge to make predictions, but only the virology module is allowed to *learn* from that text. The general-purpose weights are temporarily frozen.1

The consequence is that virology knowledge accumulates in the virology module rather than diffusing across the whole network. After training, the module can simply be deleted, and the capability goes with it. Or it can be left in place for trusted deployments, when virology knowledge is needed. The knowledge can be tailored very specifically to the type of deployment needed: in our experiments, we defined four dual-use categories, so that one training run with GRAM yielded a model that can be configured sixteen different ways (“on” or “off” for each of the four categories).

## Testing GRAM

We tested GRAM in three settings of increasing realism.

First, on a synthetic dataset of children\`s stories tagged by topic, a small GRAM model could be reconfigured to “forget” any chosen topic, and each configuration performed almost identically to a separate model trained from scratch with that topic filtered out. That is, for the cost of training a single model, we achieved results that would normally require multiple training runs on different datasets.

Second, we trained a larger model on a realistic mix of web text, code, and scientific papers, with four dual-use domains: virology, cybersecurity, nuclear physics, and a niche programming language (to serve as a proxy for specialized dual-use code). The capability associated with each dual-use domain is routed to its own module. Deleting a module removed the corresponding capability about as effectively as never having trained on that data at all. Remarkably, we find that this removal did not degrade general performance.

We also tested whether an attacker could recover the removed knowledge by training on a small amount of malicious data; GRAM resisted this about as well as data filtering did. By contrast, an “unlearning” technique applied after training only *suppressed* the knowledge—it was easy to restore with a small amount of fine-tuning.

Third, we ran the experiment at seven model sizes from 50 million to 5 billion parameters. GRAM matched the performance of data filtering at every size, and the gap between “module on” and “module off” grew wider as models got larger. In terms of compute costs, attempting to bypass our protections became relatively more difficult and expensive as we scaled.

## Conclusions

As AI companies train more capable models, the need to limit access to dual-use capabilities will increase. Today, companies limit access through classifiers and refusal training. However, these safeguards are difficult to make robust without degrading performance on harmless requests. Methods like GRAM offer a potential path toward access control that is more robust.

This is early research, and there are clear limitations. We haven\`t tested GRAM at frontier scale or in a production training pipeline. (As noted above, it hasn\`t been applied to any of our Claude models.) Our evaluations quantify performance in terms of next-token prediction ability, rather than performance on real downstream tasks. And there\`s a deeper open problem that applies to data filtering and methods like GRAM: some dual-use capabilities might be so entangled with general knowledge that no method can separate them cleanly.

*For further details on our experiments, read the [post](https://alignment.anthropic.com/2026/modular-pretraining/) on our Alignment Science blog.*

#### Footnotes

1\. One technical detail is that the virology module is also sometimes turned on when learning from general-purpose text. We find this helps the modules “work together” more effectively.

[Share on Twitter](https://twitter.com/intent/tweet?text=https://www.anthropic.com/research/off-switch-dual-use)[Share on LinkedIn](https://www.linkedin.com/shareArticle?mini=true&url=https://www.anthropic.com/research/off-switch-dual-use)

## Related content

### A global workspace in language models

New interpretability research reveals an emergent mental workspace in Claude that holds internal thoughts that don\`t appear in the model\`s output.

[Read more](https://www.anthropic.com/research/global-workspace)

### Anthropic Economic Index report: Cadences

In our latest Economic Index report, we sample hourly for the first time to ask: When do people come to Claude? What do they produce with it? And how do they perceive AI's impact on their work?

[Read more](https://www.anthropic.com/research/economic-index-june-2026-report)

### Project Fetch: Phase two

We report results from our latest test of whether Claude can help Anthropic employees perform sophisticated robotics tasks. We found that Claude Opus 4.7, operating without human assistance, was about 20 times faster than the fastest human team at all tasks completed by participants less than a year ago.

[Read more](https://www.anthropic.com/research/project-fetch-phase-two)

[Return to homepage](https://www.anthropic.com/)

### Products

- [Claude](https://claude.com/product/overview)
- [Claude Code](https://claude.com/product/claude-code)
- [Claude Code Enterprise](https://claude.com/product/claude-code/enterprise)
- [Claude Cowork](https://claude.com/product/cowork)
- [@Claude](https://claude.com/product/tag)
- [Claude Design](https://claude.com/product/design)
- [Claude Science](https://claude.com/product/claude-science)
- [Claude Security](https://claude.com/product/claude-security)
- [Claude for Chrome](https://claude.com/chrome)
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

### Claude Platform

- [Overview](https://claude.com/platform/api)
- [Developer docs](https://platform.claude.com/docs)
- [Pricing](https://claude.com/pricing#api)
- [Ecosystem](https://claude.com/ecosystem)
- [Marketplace](https://claude.com/platform/marketplace)
- [Regional compliance](https://claude.com/regional-compliance)
- [Claude on AWS](https://claude.com/partners/claude-on-aws)
- [Google Cloud](https://claude.com/partners/google-cloud-vertex-ai)
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
- [Plugins](https://claude.com/plugins)
- [Powered by Claude](https://claude.com/partners/powered-by-claude)
- [Service partners](https://claude.com/partners/services)
- [Tutorials](https://claude.com/resources/tutorials)
- [Use cases](https://claude.com/resources/use-cases)

### Programs

- [Startups](https://claude.com/programs/startups)
- [Research Labs](https://claude.com/programs/claude-team-plan-for-research-labs)

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
