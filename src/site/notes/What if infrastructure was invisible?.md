---
{"dg-publish":true,"permalink":"/what-if-infrastructure-was-invisible/","created":"2026-02-07T22:58:25.581-08:00","updated":"2026-03-01T17:38:21.264-08:00"}
---

When we were kicking around ideas in 2023, [Ross](https://rosslazer.com/) asked this question: what if the code didn't need to know it was running on the cloud?

Like truly write once, run everywhere. Some magic happens and you get durable data structures, blob-storage file systems, and all the other stuff that makes up a real application. But the cloud translation is automatic. You just write code.

Recently I decided to actually explore it. 

My thesis going into it was: in any language, where you declare a variable tells you how long it should live. And the language itself is the contract, especially the standard library.

A variable at the top of a file? That's meant to outlive a single request: it's state. A variable inside a function? That's scratch work: it dies when the function returns. Developers already express infrastructure intent through scoping. So why not infer the infrastructure from it?

Once you see it that way, the mapping feels like it can work:

| Stdlib Primitive                         | Inferred Infrastructure          |
| ---------------------------------------- | -------------------------------- |
| `node:http` / `createServer`             | Container endpoint               |
| `node:sqlite` / `DatabaseSync`           | Durable SQL                      |
| `new Map()` (module scope)               | Durable KV                       |
| `new Array()` (module scope, push/shift) | Durable queue                    |
| `node:fs` / `writeFile`                  | Blob storage                     |
| `EventEmitter` (module scope)            | Distributed pub/sub              |
| `setInterval(() => fetch(...))`          | Durable cron                     |
| `process.env.*`                          | Environment / Secrets management |

I built a working prototype with Claude in a couple days to prove this out. You can see it [here](link).

Then I kept pulling the thread... and realized the thing that makes this possible is the same thing that makes it unnecessary.

I was building an inference engine to figure out what infrastructure code needs. But when code is AI-generated, the AI already knows. It decided that Map should be durable when it wrote it. It doesn't need an inference engine to tell it what it already intended. The cost of writing code is approaching zero. The cost of configuring infrastructure is right behind it.

With AI writing millions of lines of code, the "developer tools" audience is less the developer and more the AI. And the gap between "AI can figure out the infra" and "AI can deploy the infra" is closing in months, not years. Claude Code can already run shell commands. MCP servers can talk to cloud APIs. The agent will just ask "AWS or GCP?" and handle the rest.

Even the portability argument dissolves. "Write once, run anywhere" was the answer to "rewriting is expensive." But rewriting isn't expensive anymore. If you want to move from Vercel to AWS, an agent can do that migration in days, not months.

So that's where I'm going to leave it. The design docs and prototype are open source if you want to dig in. If this is interesting to you, I'd love to hear from you.