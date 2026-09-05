---
{"dg-publish":true,"permalink":"/notes/checking-distributed-invariants-with-waldo/","created":"2026-09-05T13:27:58.336-07:00","updated":"2026-09-05T14:35:29.461-07:00","dg-note-properties":{}}
---

I learned about Jim Waldo et al's [note on distributed computing](https://waldo.scholars.harvard.edu/publications/note-distributed-computing) while building out [invisible](https://github.com/nickdirienzo/invisible) back in Feb 2026. `invisible` tried to infer distributed semantics from local code. `waldo` starts with a different hypothesis: the deployment model is part of the program's semantics, so correctness checks need to see both.

In my career, I've seen engineers overlook the physics of distributed systems (myself included!). You encounter these problems with a service-oriented architecture, but even one application running across multiple replaceable processes is enough.
## This worked locally
The other day at work a coding agent took a crack at building a feature and included deferred work via `setTimeout`:

```typescript
 const timer = setTimeout(async () => {
    await updateExpiration();
  }, delayMs);

  timer.unref();
```

This works locally. In production, the service runs in a container whose lifetime is controlled by an orchestrator. A deployment or failure can destroy the process before the callback runs.

A conventional lint rule could ban `setTimeout`, but that would be wrong: process-local deferred execution is totally okay for some use cases and entirely valid in certain deployments. `waldo` can make the rule conditional on the deployment semantics and surface the risk so the author can accept it, mark it as a false positive, or fix it.
## Analyzing code _and_ deployment
That observation led me to build [`waldo`](https://github.com/mirage-security/waldo). It's a static analysis tool that treats source code and deployment configuration as two sources of architectural facts, then joins them using deterministic policies. 

A process-local timer is one fact. A restartable deployment is another. Neither is a bug alone; together they may violate an invariant. 

Assume the above code is shipped as a two-replica Kubernetes Deployment. All you need to do is configure `waldo.yaml` to point at your deployment configuration and the tool extracts deployment facts from it:

```yaml
version: 2
service: expiration-worker

artifacts:
worker:
  entrypoint: src/index.ts

deployments:
production:
  artifact: worker
  from:
	adapter: kubernetes
	source: deploy/production.yaml
	resource: Deployment/expiration-worker
	with:
	  namespace: production

recommendations:
non-durable-deferred-execution:
  instruction: For durable deferred execution, we typically use a Temporal workflow.
```

In this example, `waldo` reports:

```
Analysis: 1 provider completed; 1 code fact; 1 deployment; 4 policies.
    javascript: complete coverage; 1 code fact; 1 files attempted
    expiration-worker/production: deployment adapter kubernetes emitted 5 facts

  WARNING non-durable-deferred-execution This deferred work may never run if the process stops or restarts first.
  src/index.ts:1 [unresolved]
    code evidence (javascript deferred-execution "timer"): correctness.criticality="unknown",
    execution.authority="process-local"
    deployment evidence (expiration-worker/production): process.restartable=true,
    scheduling.processLocal.durable=false
    recommendation: For durable deferred execution, we typically use a Temporal workflow.
    finding: waldo:v2:ff6627d0f18a688c41132f825d8fa06d50f4c5be712df5788e4277a9d6e5bbb0

  1 findings: 1 unresolved, 0 accepted, 0 false-positive; 0 failing
```

I backtested `waldo` against the agent’s initial commit. It found this exact issue and steered the agent toward the durable technology we already use.
## Why not just put this in `AGENTS.md`?
You could put this guidance in `AGENTS.md` and ask a coding agent to remember it. But that makes correctness depend on the model noticing the relevant instruction, finding the relevant deployment context, and reasoning about the interaction correctly every time.

If the rule can be derived from facts we already have, I'd rather make that part deterministic. The source says the work is process-local. The deployment says the process is restartable. A policy can join those facts the same way every time, whether the code was written by a human or an agent.

The agent is still useful where judgment is actually required: understanding the finding, choosing an implementation, and making the change. It just doesn't need to rediscover an architectural invariant that we already know how to evaluate statically.

The source and deployment facts were extracted statically and joined by a deterministic policy. `waldo` identified the architectural conflict, our configuration supplied the preferred primitive, and the agent remained responsible for the implementation.
## What I learned
`invisible` started from the hope that infrastructure could disappear. `waldo` helped me make explicit a lot of what I'd mentally internalized over the years: distributed systems semantics can't disappear, but maybe more of their assumptions can become explicit, checkable, and hard to accidentally violate.
[]()
(PS: expect a follow-up soon because infrastructure can become invisible in specific deployment models.)