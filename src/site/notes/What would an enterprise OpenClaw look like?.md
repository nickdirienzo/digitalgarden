---
{"dg-publish":true,"permalink":"/what-would-an-enterprise-open-claw-look-like/","created":"2026-03-01T19:36:48.164-08:00","updated":"2026-03-01T20:34:57.017-08:00"}
---

Clearly my brain is obsessing over the Claw ecosystem this weekend since this is the 3rd post in 48 hours about this topic. 

While hacking on NonnaClaw, I couldn't help but think about what the multi-tenant SaaS architectural variant could look like. There are so many trade-offs to make when building software, but in the enterprise I've found two things to broadly be true:

1. Encrypting customer data on a per-tenant basis as the default; it is the customer's data. What's left as cleartext is metadata for routing and authorization.
2. Enabling customers to have the option for data sovereignty (e.g. bring your own bucket, bring your own key, run in their cloud, etc)

If I simplify Claw applications, they are primarily workflow orchestrations dressed up with a familiar UX (WhatsApp, etc). In addition to the data needs, the architectural requirements are:

**Durable orchestration.** API requests to LLMs can fail, so you'll want retries, and agents can lean on workflow orchestration to perform work instead of running it long-lived in its own container. Instead of agents spawning subprocesses and passing data through files, it would use something like Temporal, Inngest, or Dapr. These workflow orchestrations also provide polling mechanisms built-in so cron moves to the application level.

**Runtime isolation.** Given the non-determinism of agents, they must be treated as [untrusted](https://nanoclaw.dev/blog/nanoclaw-security-model). This means each agent should run in a Docker container or v8 isolate that cannot interfere with other user requests. Kubernetes is the obvious winner here due to the ad-hoc nature of running new agents, unless v8 isolates work out which would reduce the infra footprint.

**Agent capability scoping.** Building on the above, not only do agents need to be run in isolation, each agent has a separate set of allowed tools and APIs. Because they are untrusted, they should not see credentials for tool use (see [IronCurtain](https://www.provos.org/p/ironcurtain-secure-personal-assistant/) and [NonnaClaw](https://nickdirienzo.com/why-aren-t-claw-skills-just-mcp-server-install-instructions/) for the single-user version of this). This requires some sort of explainable policy engine like OPA or Cedar; explainability matters here because when agents are doing real work on behalf of humans, customers need to audit exactly what an agent was and wasn't authorized to do. While runtime isolation prevents escape, this prevents unauthorized use.

**Cloud-native IO.** Local Claws lean on the filesystem for agent work and Node polling loops for data ingestion. This local IO model breaks when you have multiple tenants on shared infrastructure. Files become per-tenant blob storage (e.g. S3). Polling loops become event-driven ingestion: instead of 500 tenants each running their own cron, a service exposes public endpoints per customer that push events into the durable orchestration layer or are centrally scheduled in that layer in batch.

**Secret management.** Claws have a massive integration surface area. These cannot be all driven by environment variables. Instead each customer will need their own secret at runtime, that only they can configure. This causes a requirement for a technology like Hashicorp Vault, OpenBao, or secrets in the database using data encryption keys from the customer key.

**Execution observability.** LLMOps is core to the product for debugging and improvement. You'll need to trace the full lifecycle of an agent run: what did the user ask, what did the agent do, what happened in the tool loop—using customer keys to decrypt and inspect the interaction. Long-term, the same data feeds analytics and quality feedback loops that make agents better over time. Given the enterprise data requirements above, this cannot just be piped to a random LLMOps startup. Ideally, this data belongs in a storage layer that supports both per-tenant encryption and analytical queries: an Iceberg lakehouse with per-namespace KMS keys or a warehouse like Snowflake or BigQuery with per-dataset CMEK; but this could be as simple as S3 with Athena.