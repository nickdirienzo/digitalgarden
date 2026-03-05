---
{"dg-publish":true,"permalink":"/post-ideas/reflecting-on-harness-engineering-lessons-from-engineering-with-ai-since-2023/","created":"2026-03-05T12:53:50.049-08:00","updated":"2026-03-05T15:05:25.835-08:00"}
---

I learned about [OpenAI's Symphony project](https://github.com/openai/symphony) today. It is really impressive and reminds me of the experiment I was working on in January with [inc](https://github.com/nickdirienzo/inc). Symphony and OpenAI's post on [Harness Engineering](https://openai.com/index/harness-engineering/) is making me move from the headspace of "experiment" to "how can I bring these capabilities to our daily development without a new tool."

I skimmed through the post when it released last month, but didn't deeply sit with it until now. As a long-time platform engineer, I agree with a lot here; these practices are what platform teams have encouraged for decades: shift left with fast feedback loops. 

While building Mirage, we developed much of the product with AI assistance before coding agents were a thing -- we were pasting code into the OpenAI playground and getting code back. Today, it is a 100x better experience than in 2023, thanks to tools like Claude Code and Codex.

Because of that experience, our repository has continued to become more legible for our coding agents. I view our repo to be 40% aligned with the harness engineering philosophy. This is my reflection on what we do well, where we can improve, and where I disagree with OpenAI's approach.
## What we do well
**[Architecture Decision Records](https://adr.github.io/madr/) (ADRs).** I'm surprised OpenAI's post doesn't mention them. We have 40+ ADRs in `doc/architecture/decisions/`. Before agents, ADRs were aspirational: the team agreed they were valuable but couldn't sustain the discipline of writing them. With agents, ADRs become a flywheel: the agent writes the ADR as part of implementing the decision, and future agents reference those ADRs when building features that touch the same surface. We regularly see Claude cite an ADR unprompted when making an implementation choice. If you've appreciated ADRs before but couldn't keep up, having agents write and leverage them is a beautiful thing. 

**Committed feature plans.** We commit specs to `doc/features/` alongside the code. This gives PR reviewers (both human and agentic) insight into the high level vision and architecture, and whether the code aligns with the plan. We should be doing more of this -- today only some features get committed plans. The goal is all of them.

**Type-safe API contracts.** We use `ts-rest` with Zod schemas for every endpoint. This is a mechanical invariant: the agent literally cannot ship an API change that doesn't match the contract. This is the kind of guardrail the post advocates: enforce the boundary, not the implementation.

**So much observability.** For an early stage startup, we have a lot of telemetry. We invested heavily in OpenTelemtry traces, logging, and metrics upfront. We ship OTel data to Observe, which our coding agent can query via MCP. This is still heavily human-in-the-loop given the complexity of the data, but even partial agent access to o11y shortens the diagnosis loop. Before agentic programming, this investment already paid for itself; now it's becoming a foundation for agent-driven debugging.

**Structured logging conventions.** We have a specific logging format (`[Component.method] Action`, context object, `serializeErrorLike()` for errors). This is a pattern agents still violate, which tells me it needs to move from documentation to enforcement (a linter rule, not a paragraph in CLAUDE.md). Fortunately, GritQL makes this a lot more achievable and reduce the feedback loop for agentic development.

**Agent-driven code review.** We have a Claude Code GitHub Action workflow that reviews every PR against our team's standards: type safety, multi-tenant data scoping, logging patterns, and security. It catches things humans miss because it never gets tired of checking the checklist. We've biased our prompt to focus on getting changes out the door while still maintaining security and quality. I discovered [Warden](https://warden.sentry.dev/) recently which threads the needle nicely between file-path determinism and skill execution, but maybe the table of contents approach obviates the need for this.

**Fast CI/CD to production.** We have strong CI/CD practices that enable a rollout to production within 15 minutes. This matters more in an agent-driven workflow because throughput increases. More changes shipping means you need faster recovery, not more gates. The confidence to let agents ship comes from knowing you can recover quickly, not from preventing every possible mistake upfront.
## Where we can improve
**Our CLAUDE.md is the content, not the table of contents.** It grew to 500+ lines and agents started ignoring patterns buried in the middle. OpenAI's approach of a short AGENTS.md (~100 lines) that points to deeper domain docs is right. "Too much guidance becomes non-guidance" is something we experience daily: the agent blatantly ignores patterns that seem simple to follow, like using our error serialization utility or response handling helpers.

**Agents can't boot our stack yet.** We use PM2 for local dev, but the agent can't launch the app, drive it with a browser, and validate its own changes. It's working on static code, blind to whether the change actually works. This is the single biggest gap between where we are and the full harness engineering vision.

**Documentation is better than enforcement, but enforcement is what works.** If agents keep violating a pattern, it's not a documentation problem: it's an enforcement problem. We need to move more conventions from prose in CLAUDE.md into automated guardrails (GritQL and tests). A lint error the agent has to fix is worth more than a paragraph it might ignore.

**Some ADRs have drifted.** We have 40+ ADRs but some represent completed transitions where the "decision" is now just "how it works," and some foundational early decisions were never recorded. We need a doc-gardening process, which is something OpenAI does with recurring agent jobs, and we should too.
## Where we disagree
**PRs per engineer per day is an incomplete metric.** OpenAI reports 3.5 PRs/engineer/day. I'm doing 5+ with Claude Code, with one or two more complex explorations running in the background. We use this metric too since each PR is effectively a feature (+137% output in a week compared to the previous 3 months) -- it loosely measures how frequently the system is changing. But it doesn't capture how much complexity is being added or removed. 

**Agents should have access to more context sources.** OpenAI deliberately keeps agents out of Slack. I understand the reasoning -- Slack is noisy and unstructured -- but I'm not sure why you'd limit the agent's access to context that helps it do its job. We give Claude access to Metabase and Notion via MCP so it can read task and customer context. The repo is for execution; the project management tool is for "why this matters to the business" -- where the rest of the business works.

**Distinct o11y stacks per worktree is overkill.** OpenAI runs separate observability for each worktree. You can get the same result by adding a worktree identifier to your OTel resource attributes and filtering in your existing APM. Same data, no extra infrastructure. I'd love to know how they landed on independent stacks.

**Recurring invariant jobs are interesting, but I'd start per-PR.** Their "golden principles" are enforced by recurring sweeps rather than per-PR gates. They don't explain why and instead just shrug it off saying . I think the answer is throughput at their scale -- per-PR enforcement is synchronous and blocks the agent loop. But for a startup, start with per-PR enforcement in CI and graduate to recurring agents as volume increases. You're not at the scale where the blocking cost matters yet.
## What's Next For Us
We're working on closing the feedback loop. The end state is: given a single prompt, the agent can reproduce a bug, record a video, implement a fix, validate the fix by driving the application, record a second video, and open a pull request with evidence. Escalate to a human only when judgment is required.

We're not there yet, but each step gets closer. And none of it requires a dedicated platform team. I'm working on a companion post with the concrete checklist for other startups.
