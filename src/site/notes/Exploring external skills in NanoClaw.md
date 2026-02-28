---
{"dg-publish":true,"permalink":"/exploring-external-skills-in-nano-claw/","created":"2026-02-28T09:34:02.709-08:00","updated":"2026-02-28T13:43:13.996-08:00"}
---

I read [Don't trust AI agents](https://nanoclaw.dev/blog/nanoclaw-security-model) this morning by the creator of NanoClaw ([Gavriel Cohen](https://x.com/Gavriel_Cohen)). I generally agree with this take: we shouldn't provide secrets as inputs into LLMs or have them run with full permission on the filesystem (yet!).

NanoClaw's approach to agentic routing is using containers for isolation as a foundational primitive. This provides hardening and guardrails that are hard to workaround. I think this is a great project and have started to use it personally in my homelab.

I left a [small comment](https://news.ycombinator.com/item?id=47196849) on the HN thread, but I think this is hard to articulate in short form. I tried, but definitely wasn't able to share my thoughts well enough, so here they are. I don't mean any of this as criticism of NanoClaw, but more of a thought experiment of how can we make NanoClaw the backbone of agentic personal assistants safely.

## The problem with the current skills contribution model

What surprises me is that despite all of the focus on security, there's still a long-term sprawl of code via the "features as skills" contribution model. These contributions aren't _just_ skills. They are instructions for Claude to modify NanoClaw's critical code paths to support the new capability. 

While the project says that the codebase is small enough for someone to understand in an afternoon. This will not be true for someone adding a ton of skills that have modified the critical code paths of the fork, the SQLite database shared by all capabilities, and keeping each personal fork up to date with upstream. 

From a patch management perspective, I'm a bit worried too. Since every fork is personal it means that you need to have a trusted agent watching dependencies for vulnerabilities and updating appropriately. There's no version of the skill since it's intertwined with the core codepaths of NanoClaw.

## Why spend tokens on modifying the code?

The AI-native response to this is: just let Claude Code handle it. I agree that this will work in almost all cases, but my question is: why are spending tokens on code modifications instead of using proper codemods or better yet, allowing NanoClaw to properly be a framework that supports a plugin architecture. 

Plugin architectures are not new. We've had them for decades that allows for simple interop between the framework and external functionality. Think like:
* Browser extensions: browsers provide ways for other code to run safely while interoperating with the user experience cleanly.
* VSCode extensions: they don't modify the core code of VSCode, but they augment capaibilties. 
* Terraform providers: Terraform core doesn't know about AWS; the AWS provider does.

## Skills as config, not code

The agent ecosystem already has a ton of "plugins" readily available with proprietary and community MCP servers. NanoClaw needs to know how to interoperate with them. 

What if skills were just a manifest, a small inbound script, and a set of MCP capabilities defined as config? This could look something like this:

1. A `skill.json` manifest declaring which community MCP servers to use for outbound (sending messages, etc.)
2. A small standalone entrypoint script (~20-50 lines) that handles inbound (polling an API, writing events to an inbox directory)
3. A `SKILL.md` with instructions for how to set it up and integrate into the runtime. 

This avoids every skill modifying the SQLite database and the core event loop, which risks each one stepping on each others' toes.
## Skills don't need to live in the repo

If all of this can work, this would mean that NanoClaw's contribution becomes primarily an event router, a container orchestrator for agents, and an interface for writing into inbox/outbox. 

Taking the idea to a further extreme, this unlocks the ability for skills to be external to the NanoClaw repository entirely. This would allow skills to be their own GitHub repos, just like MCPs with Claude Desktop.

It creates a new problem of: how does NanoClaw "install" a remote skill? But we can lean on a skill for this, or provide it as a simple CLI command: `nanoclaw install`.

## Trying all of these ideas out

I felt like I had a clear picture of what this could look like, so I set Claude off on an adventure. Could we take the ideas here and actually make it work? The criteria being: we are able to replicate the end to end onboarding workflow with WhatsApp without having WhatsApp in the codebase at all. 

That would give us integration one. If that works, we can try with Gmail, and then we can try with another random skill. Ideally all of these are backed by popular or official MCP servers instead of a ton of custom code

### Problem 1: MCP on its own is insufficient

Containerized agents in NanoClaw are only responsible for thinking. Not for doing. Any action goes back to the host process to execute outbound messages. 

This means that skill contributions can't only have an inbound entrypoint script. They need to also provide an outbound handler script. Ideally we should be able to provide the containerized agent a subset of MCP capabilities and scope, i.e. don't send a message to every group, just the one you are registered to talk with.

I will have to come back to this.