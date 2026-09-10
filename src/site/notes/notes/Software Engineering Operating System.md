---
{"dg-publish":true,"permalink":"/notes/software-engineering-operating-system/","created":"2026-09-10T16:36:25-07:00","updated":"2026-09-10T16:36:25-07:00","dg-note-properties":{}}
---

As an industry, we've been talking a lot about the [dark factory pattern](https://www.danshapiro.com/blog/2026/01/the-five-levels-from-spicy-autocomplete-to-the-software-factory/).

As I've been building out the foundations for Mirage's own dark factory, I've started to think factory is the wrong noun for the underlying system. A factory describes an autonomous delivery system: inputs go in and some desired output comes out. What we're actually building underneath it is the platform that lets many different systems and processes run, interact, and evolve independently.

OpenAI's recent [Defense Factory](https://openai.com/the-defense-factory/) made me start questioning whether factory is the right noun for the underlying system. It's essentially the same idea as a software factory, but applied to security remediation.

The input is different. Instead of starting with a feature spec, it starts with a vulnerability. The acceptance criteria is different too: the vulnerability is no longer exploitable in our software system.

But both workflows ultimately need to produce the same thing: a change.

And once that change exists, it enters the same lifecycle as any other change: test, verify, authorize, deploy, and observe. The systems have different bounded responsibilities and acceptance criteria, but they're both trying to safely move the software system from one state to another.

So I don't think these are really separate factories in the architectural sense. They're different autonomous systems made up of processes that ultimately depend on the same underlying platform.

You know what else is built around autonomous processes? Operating systems.

An operating system schedules processes, gives them access to resources, isolates them from one another, manages identities and permissions, provides mechanisms for communication, and manages their lifecycle when things fail. A process doesn't need to bring its own scheduler, memory manager, or process supervisor; those are provided by the operating system beneath it.

I think autonomous software engineering needs the same thing, but across distributed systems.

A vulnerability remediation system shouldn't need to invent its own execution infrastructure, authorization model, deployment machinery, or verification system. Neither should a feature-development agent, dependency updater, code reviewer, or pentesting system. They should be able to run independently while sharing a common platform for scheduling work, executing it with the right capabilities, exchanging results, and safely turning proposed changes into running software.

Bringing this down from theory to practice, I'm describing durable workflow orchestration. Once you have that platform, the rest starts to look like operating systems primitives: scheduling, execution, identity, isolation, communication, and failure recovery.

Of course autonomous workflows will be developed independently and in parallel, but they need a coherent platform underneath them. That's what I mean by a Software Engineering Operating System: a shared control plane for coordinating the processes that build, test, secure, deploy, and operate software.

With this lens, the things we call "factories" sit on top of the Software Engineering Operating System. A Defense Factory is one workflow. A Software Factory is another. Dependency maintenance, code review, pentesting, and deployment are others.

Maybe I've been working on platform-engineering-shaped problems for too long, but none of this feels fundamentally new. We've been building pipelines and automating pieces of the software delivery lifecycle for decades. What's changing is that more of the processes participating in that lifecycle can now operate autonomously.
