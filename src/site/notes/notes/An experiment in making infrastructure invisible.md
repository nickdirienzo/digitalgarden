---
{"dg-publish":true,"permalink":"/notes/an-experiment-in-making-infrastructure-invisible/","created":"2026-02-07T22:58:25.581-08:00","updated":"2026-09-04T22:27:53.983-07:00","dg-note-properties":{"tags":null}}
---

I started writing this earlier in the year (Feb 2026), and put it down for many months. I wish I posted this while I was building but I was disappointed with the result so I decided to not post it. Anyway, I realized I should just post it because that's what a digital garden is about.

---

When we were kicking around ideas in 2023, [Ross](https://rosslazer.com/) asked this question: what if the code didn't need to know it was running on the cloud?

Like truly write once, run everywhere. Some magic (compilation?) happens and you get durable data structures, blob-storage file systems, and all the other stuff that makes up a real cloud application.

I explored it recently under this hypothesis: in any language, where you declare a variable tells you how long it should live, and through that language's standard library, you can determine the deployment contract automatically. Seems easy enough to falsify.

A variable at the top of a file is state. A variable inside a function is ephemeral; it dies when the function returns. If developers already express infrastructure intent through scoping, can we infer infrastructure with it?

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

I built a [research proof of concept](https://github.com/nickdirienzo/invisible) with Claude in a couple days to prove this out. It mostly worked: a hello world became a container, `process.env.STRIPE_SECRET_KEY` became a managed secret, a `setInterval` became a cron job, and a module-scope `Map` became a Valkey hash without the code ever mentioning Valkey. 

Then I wrote a counter: read the `Map`, add one, write it back. On one process it's fine. On two replicas it drops increments, and nothing in the source tells you that. The code was correct locally and wrong in a HA setting, which is exactly the difference I was trying to make disappear.

And that is running head-first into what I've started calling [Waldo's Wall](https://waldo.scholars.harvard.edu/publications/note-distributed-computing). You can give local and remote things the same interface, but you can't give them the same semantics. I loved my Distributed Systems and Operating Systems courses in university, so I probably should have seen this coming. But I didn't want to believe it was real; I felt like I could engineer around the problem. 

Clearly, I'm a bit stubborn. But at least I tried. I learned some new things. And I learned about a really great paper on distributed computing in 1994 that still applies in 2026.