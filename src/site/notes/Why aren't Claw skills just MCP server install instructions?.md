---
{"dg-publish":true,"permalink":"/why-aren-t-claw-skills-just-mcp-server-install-instructions/","tags":["essay"],"created":"2026-03-01T10:32:39.820-08:00","updated":"2026-03-01T11:51:22.361-08:00"}
---

I've been thinking about claws since OpenClaw released. I think we all have (e.g. [Karpathy's tweet](https://x.com/karpathy/status/2024987174077432126)).

They're powerful, fun, and scary. I set up [NanoClaw](https://nanoclaw.dev/) because I really liked its security model: put the non-determinism in a container so it can't wreck your system. If you're unfamiliar, NanoClaw skills are Claude Code instructions that rewrite your fork's source code to add capabilities. 

I couldn't stop thinking that these skills are just plugins, but without any of the guarantees we expect from plugin architectures. No typed interfaces, no versioning, no supply chain scanning. What if the host didn't need to know about the skill's code at all and could still provide the new capabilities? I built [NonnaClaw](https://github.com/nickdirienzo/nonnaclaw) to test that idea: an experimental fork of NanoClaw with a rebuilt skill layer that treats MCP servers as the capability boundary. I [wrote about the process here](https://nickdirienzo.com/exploring-external-skills-in-nano-claw/).
## Why spend tokens on code changes?
This was the first thought that led me down this rabbit hole. I didn't fully believe that personalized forks were better than a stable platform that provides plugins. What I actually wanted was determinism and a stronger security model.

That led me to the question: can these skills just be MCP setup instructions? An MCP server is a normal software project. It has dependency management, versioned releases, typed interfaces, and it can be scanned for vulnerabilities the same way any package can. A skill that teaches an LLM to construct curl commands at runtime has none of that. The execution is non-deterministic, the interface is untyped, and there's nothing to scan because the "code" is regenerated from markdown on every invocation.

We have plugin architectures for a reason. VSCode extensions don't patch `editor.main.js`. They operate within a typed API that the host exposes. The extension gets capabilities without changing the application's code. NanoClaw skills rewrite the host. OpenClaw skills prompt-inject the host. Neither is a plugin architecture.
## Putting this to the test
To keep the code simple and small, NonnaClaw separates the three layers:
- **Capabilities:** These are provided as tool calls to MCP servers. Typed, versioned, maintained by someone else. Normal software.
- **Configuration:** a `skill.json` manifest declaring what to spawn and how to scope requests. Effectively a flexible authorization layer on the tools themselves for the agent.
- **Workflows:** orchestration skills that reference tools abstractly. Can't execute without the MCP tools present.

A skill is its own repo with two files. [nonnaclaw-whatsapp](https://github.com/nickdirienzo/nonnaclaw-whatsapp) wraps a community WhatsApp MCP server with per-group scoping. [nonnaclaw-github](https://github.com/nickdirienzo/nonnaclaw-github) wraps GitHub's official MCP server (`ghcr.io/github/github-mcp-server`) via Docker with read-only scoping: every write operation (`merge_pull_request`, `push_files`, `delete_file`, etc.) is `allow: false`. The `SKILL.md` runs once during install to walk through auth. After that it's out of the picture.

```json
{
  "name": "whatsapp",
  "mcp": {
    "command": "uv",
    "args": ["--directory", "./whatsapp-mcp/whatsapp-mcp-server", "run", "main.py"],
    "pollTool": "list_messages"
  },
  "scopeTemplate": {
    "send_message": { "allow": true, "scopedParams": ["recipient"] },
    "list_messages": { "allow": true, "scopedParams": ["chat_jid"] },
    "list_chats": { "allow": false }
  }
}
```

The host spawns the MCP server, bridges it to HTTP, polls for inbound messages. The agent connects through a scoping proxy that enforces per-group authorization. The agent discovers typed tools through the protocol: names, descriptions, input schemas. No prompt injection needed.

After install, the skill dissolves. It's just infrastructure.
## Does this apply to OpenClaw?
Since the system worked end-to-end, I started wondering: does this pattern work for OpenClaw?  OpenClaw skills work differently than NanoClaw skills, but the same layers (capabilities, configuration, workflows) are conflated.

An OpenClaw skill is a `SKILL.md` that gets injected into the agent's system prompt at runtime. The agent reads the instructions and generates bash commands, API calls, or code on the fly. No typed interface. The execution model: unversioned markdown → prompt injection → LLM generates arbitrary code → exec with agent privileges.

I looked at a handful of popular skills and was surprised to see the same pattern over and over again:
* **Notion:** teaches the agent to construct `curl` commands against the Notion API. Three competing versions on ClawHub (`notion`, `notion-skill`, `better-notion`) all reimplementing the same thing. Notion ships an official MCP server (`@notionhq/notion-mcp-server`) with typed tools and scoped permissions.
* **GitHub:** wraps `gh` CLI. There's a GitHub MCP server on the official Docker registry.
* **summarize:** wraps a real Homebrew binary with versioned releases. The capability is already packaged software. The skill just tells the LLM to generate bash commands to call it.

For every major skill I looked at, typed MCP servers already exist. It feels like the skills are worse versions of interfaces that are already out there.
## Understanding the current security risks with claws
ClawHub skills already ship with code. They have install scripts, `package.json` files, versioned releases, changelogs, metadata schemas declaring required binaries and env vars. It's like 80% of a package manager, but then used prompt injection for the last 20%.

This is why NanoClaw runs agents in containers with limited access. You don't have to worry about this risk vector because the agent doesn't have host access.

Snyk scanned ~4,000 ClawHub skills and found [36% had prompt injection vulnerabilities](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/). The [ClawHavoc campaign](https://www.koi.ai/blog/clawhavoc-341-malicious-clawedbot-skills-found-by-the-bot-they-were-targeting) documented 824 malicious skills. The attack pattern is simple: a skill looks legitimate, but its "Prerequisites" section tells you to download and run an executable. The barrier to publishing: a SKILL.md and a week-old GitHub account.

An MCP server published to npm or PyPI gets the same supply chain scrutiny as any other package. It can be scanned by Snyk, Dependabot, Socket. It has typed exports that can be audited. A SKILL.md is a markdown file that an LLM interprets at runtime. There's nothing to scan because the code doesn't exist until the agent generates it.
## How the models compare
Each approach makes different tradeoffs across the same three layers:

| Layer         | OpenClaw                  | NanoClaw            | NonnaClaw                            |
| ------------- | ------------------------- | ------------------- | ------------------------------------ |
| Capabilities  | Prompt-injected bash/curl | Codemods to source  | MCP servers                          |
| Configuration | Same SKILL.md             | Same codemod        | `skill.json` manifest                |
| Scoping       | None                      | Container isolation | Container isolation + per-tool proxy |
## This is an experiment
This is early. There's still work to do: securing the host layer, strengthening the MCP proxy, and some other ideas. It's an experiment, not a product.

The install-time trust problem doesn't go away. SKILL.md runs once during setup and could instruct Claude Code to do bad things. But the blast radius is one-time versus permanent prompt injection on every agent turn. Same trust model as `npm install`.

The accessibility argument is real: SKILL.md is easier to write than an MCP server, and for non-developers, the AI _is_ the interface. But Claude Code can scaffold a working MCP server from a description in minutes. `package.json` took more work than a loose shell script too, and that tradeoff was worth making. The friction argument gets weaker every month. The security argument doesn't.

If you want to poke at it: [nonnaclaw](https://github.com/nickdirienzo/nonnaclaw) is the fork. [nonnaclaw-whatsapp](https://github.com/nickdirienzo/nonnaclaw-whatsapp) is the simplest skill example: three files, wraps a community MCP server, scoped per group. If you've built OpenClaw or NanoClaw skills and want to try converting one, I'd like to know where the pattern breaks down.