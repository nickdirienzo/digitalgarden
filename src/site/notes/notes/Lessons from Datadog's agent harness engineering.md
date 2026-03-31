---
{"dg-publish":true,"permalink":"/notes/lessons-from-datadog-s-agent-harness-engineering/","created":"2026-03-31T12:00:18.190-07:00","updated":"2026-03-31T12:39:46.175-07:00"}
---

I recently read [Datadog's blog post](https://www.datadoghq.com/blog/ai/harness-first-agents/) on how they built a Redis implementation in Rust with AI. There were a number of ideas here that resonated with how I build.

They leverage ADRs to encapsulate not just the new architectural capabilities and why, but also the properties and invariants of the system. 

They leverage ADRs to encapsulate not just the new architectural capabilities and why, but also the properties and invariants of the system. This prose is then translated by AI into a formal TLA+ specification, which feeds other verification layers: [Deterministic Simulation Testing](https://github.com/nerdsane/redis-rust/blob/08ff584bb1afd1af8f7cdab6f6737d25cfe2ed25/docs/DST_GUIDE.md) (DST), Rust-specific model checkers like [Stateright](https://github.com/stateright/stateright) and [Kani](https://github.com/model-checking/kani), and broad system telemetry checks.

This pipeline is powerful, and I think it can be leveraged for "less formal" systems building. When building features, I imagine that looking more like:

1. **Design feature implementation plan.** Include ADRs are necessary when making significant changes. Either document includes properties and invariants that must be true. 
2. **Translate properties into a test plan.** An AI coding agent translates those properties into a test plan that includes example-based unit/integration tests, as well as property-based testing (e.g. [fast-check](https://fast-check.dev/) in Node). This provides more coverage than the previous era of development at about the same speed. PBT is particularly helpful with complex systems, workflows, and algorithms that are more open-ended.
3. **Validate system behavior end-to-end.** End-to-end tests are an analog of DST for product development. They exercise real system behavior rather than isolated units. The agent can take this further by querying your o11y stack during test runs to find performance issues that assertions alone won't catch.

At Mirage, while we have much more unit/integration test coverage than I would have thought at this stage, we are now starting to push further with PBTs, especially around our more complex and open-world workflows. We've discovered a handful of edge cases through early PBTs on core algorithms, and this style of testing provides additional confidence in the changes we're making.

I'm excited to continue evolving our harness for agentic engineering.