---
{"dg-publish":true,"permalink":"/what-if-nano-claw-skills-didn-t-need-to-change-the-codebase/","tags":["essay"],"created":"2026-02-28T09:34:02.709-08:00","updated":"2026-03-01T03:13:09.252-08:00"}
---

I read [Don't trust AI agents](https://nanoclaw.dev/blog/nanoclaw-security-model) this morning by the creator of NanoClaw ([Gavriel Cohen](https://x.com/Gavriel_Cohen)). I generally agree with this take: we shouldn't trust our agents and provide safety-by-default for agentic behavior.

NanoClaw's approach to agentic routing is using containers for isolation as a foundational primitive. This provides hardening and guardrails that are hard to workaround. I think this is a great project and have started to use it personally in my homelab.

I left a [small comment](https://news.ycombinator.com/item?id=47196849) on the HN thread, but I think this is hard to articulate in short form. I tried, but definitely wasn't able to share my thoughts well enough, so here they are... and some code that shows the thesis end-to-end.
## The problem with the current skills contribution model
What surprises me is that despite all of the focus on security, there's still a long-term sprawl of code via the "features as skills" contribution model. These contributions aren't _just_ skills. They are instructions for Claude to modify NanoClaw's critical code paths to support the new capability. 

While the project says that the codebase is small enough for someone to understand in an afternoon. This won't hold for someone who's added a ton of skills that modify the core code paths, the shared SQLite database, and then has to keep their fork in sync with upstream.

From a patch management perspective, I'm worried too. Since every fork is personal it means that you need to have a trusted agent watching dependencies for vulnerabilities and updating appropriately. There's no version of the skill since it's intertwined with the core codepaths of NanoClaw.
## Why spend tokens on modifying the code?
The AI-native response to this is: just let Claude Code handle it. I agree that this will work, but my question is: why are we spending tokens on code modifications instead of using deterministic codemods or allowing NanoClaw to properly be a framework that supports a plugin architecture. 

Plugin architectures are not new. We've had them for decades, and they allow for simple interop between an application and external functionality. Think: browser extensions, VSCode extensions, and Terraform providers.
## Skills as config, not code
The agent ecosystem already has a ton of "plugins" readily available with proprietary and community MCP servers. NanoClaw needs to know how to interoperate with them. 

What if skills were just a manifest, a small inbound script (to keep fetching non-agentic), and a set of MCP capabilities defined as config? This could look like:

1. A `skill.json` manifest declaring which MCP servers to use for outbound (sending messages, etc.)
2. A small standalone entrypoint script (~20-50 lines) that handles inbound (polling an API, writing events to an inbox directory)
3. A `SKILL.md` with instructions for how to set it up and integrate into the runtime. 

This avoids every skill modifying the SQLite database and the core event loop, which risks each one stepping on each others' toes.
## Skills don't need to live in the repo
If all of this can work, this would mean that NanoClaw's contribution becomes primarily an event router, a container orchestrator for agents, and an interface for writing into inbox/outbox. 

Taking the idea to a further extreme, this unlocks the ability for skills to be external to the NanoClaw repository entirely. This would allow skills to be their own GitHub repos, just like MCPs with Claude Desktop.

It creates a new problem of: how does NanoClaw "install" a remote skill? But we can lean on a skill for this, or provide it as a simple CLI command: `nanoclaw install`.
## Trying all of these ideas out
Could we take the ideas here and actually make it work? The criteria being: we are able to replicate the end to end onboarding workflow with WhatsApp without having WhatsApp in the codebase at all. 

That would give us integration one. If that works, we can try with Gmail, and then we can try with another random skill. Ideally all of these are backed by popular or official MCP servers instead of custom code.
### Problem 1: MCP on its own is insufficient
Containerized agents in NanoClaw are only responsible for thinking. Not for doing. Any action goes back to the host process to execute outbound messages. This means skills need to handle both inbound and outbound, not just one direction.

There's also an authorization gap. MCP servers expose all their tools to any client that connects. If a containerized agent connects directly to a WhatsApp MCP server, it can send messages to anyone, not just the group it's responsible for. We need a way to scope which tools and parameters each agent is allowed to use.
### Problem 2: Global Docker container definition for agents
I skipped over this during prototyping, but each skill may need its own container runtime with OS-level dependencies. Right now, there's global sprawl across all skills instead of just on a per-skill basis. This is still an open problem. In the current architecture, MCP servers run on the host (trusted zone) so container dependencies are less of an issue for skill-specific tooling, but it's not fully solved.

I'd like to go back to this and make it so that skill MCP servers run in containers. Unfortunately it seems like many MCP servers don't provide Docker images, which feels like a gap in the ecosystem. Containers are a wonderfully simple deployment artifact, even if that deployment is your own computer.
## Forking NanoClaw
I forked NanoClaw and built [NonnaClaw](https://github.com/nickdirienzo/nonnaclaw), an experimental project that replaces the skills-as-code model with skills-as-config backed by MCP servers. I dropped the idea of an entrypoint script and instead made it work without requiring skills to make code changes. The two main additions that make this work safely are an MCP bridge and a scoping proxy, both running on the host in the trusted zone.

The bridge (`mcp-bridge.ts`) spawns MCP servers as child processes over stdio, exposes each one as an HTTP endpoint, and polls them for inbound messages on an interval. The same MCP server that handles `list_messages` for inbound also handles `send_message` for outbound. Both directions, one server, no custom channel code. (If MCP servers had Docker containers, we could simplify further.)

The proxy (`mcp-proxy.ts`) sits between the agent and the bridge, enforcing per-group authorization via `scopeTemplate` (tool allowlists and parameter pinning). For example, a family group chat can only `send_message` to its own JID. The agent never sees tools it isn't authorized for. Both run in the trusted zone for the same reason inbound does: if they ran inside the container, a compromised agent could bypass its own restrictions.

Everything else — container isolation, filesystem IPC, per-group CLAUDE.md memory, scheduled tasks, the Claude Agent SDK harness — is inherited from NanoClaw. 
## Putting it to the test
I built two external skills: [nonnaclaw-whatsapp](https://github.com/nickdirienzo/nonnaclaw-whatsapp) using [verygoodplugins/whatsapp-mcp](https://github.com/verygoodplugins/whatsapp-mcp), and [nonnaclaw-github](https://github.com/nickdirienzo/nonnaclaw-github) using Docker's official GitHub MCP server. Each with only two files: a `skill.json` and a `SKILL.md`. Similar to NanoClaw, `SKILL.md` explains how to get it integrated; `skill.json` is new to support NonnaClaw's architecture. Two MCP servers, different transports, both running through the bridge, scoped through the proxy.

I ran `/install` for both. Then I sent a WhatsApp message: "Wha was the last commit i made"

![nonnaclaw.png](/img/user/nonnaclaw.png)

After extracting skills to external packages, the code modification and channel machinery (~7,800 lines across the skills engine, Claude Code skills, and channel implementations) went away entirely. The core stays roughly the same size (about 170 line delta), with the new `mcp-bridge.ts`, `skill-registry.ts`, and `mcp-proxy.ts`. Ideally, the core now stays relatively static except for bug fixes, security patches, and runtime improvements.
## Where this goes
NonnaClaw is an experiment. [NanoClaw is the real thing](https://github.com/qwibitai/NanoClaw). NanoClaw's insight — a personal AI assistant should be small enough to understand, secure by isolation, and customizable — is the foundation everything here builds on. 

NonnaClaw explores a different answer to one question: how should skills work?

If the MCP bridge approach continues to hold up, it means any MCP server is a potential skill. The ecosystem does the hard work of building and maintaining integrations. NonnaClaw just bridges them. This could enable more flexibility for container-isolated agents, without losing the security primitives. Personally, I'm excited to have an MCP server sitting in front of my music library that Nonna can help me manage.

If you'd like to check it out, the code is at [github.com/nickdirienzo/nonnaclaw](https://github.com/nickdirienzo/nonnaclaw).