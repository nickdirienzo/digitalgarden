---
{"dg-publish":true,"permalink":"/go-to-message-queue-in-2025-for-me-it-s-temporal/","created":"2025-06-04T13:23:20.233-07:00","updated":"2025-06-04T13:45:05.134-07:00"}
---

Sadly I don't have a lobste.rs account so I can't answer [the question](https://lobste.rs/s/uwp2hd/what_s_your_go_message_queue_2025) there, but I had these thoughts and wanted to share them.

In 2025, I think most companies don't need a message queue. Message queues still have their place -- like with edge ingestion or buffering -- but I'm not sure I would reach for them for codifying business logic knowing systems like Temporal exist. 

I've been building with Temporal for over a year now in production, so maybe I drank too much durable execution Kool-Aid. 

If you're starting from scratch, a better option is to use a system like Temporal, which gives you exactly-once delivery, durability, and retries out of the box.

With a message queue, you have to handle all of those foundations yourself. Retries are easy, sure. Idempotency is a bit harder. Durable state that survives crashes and resumes from exactly where it left off? That’s where Temporal shines. It feels like magic.

The one major trade-off here is vendor lock-in. Once you start building with Temporal and really lean into it, it's really challenging to back out of. Fortunately you can self-host but operating a Cassandra stack is not very fun.

Still, for business logic, it's hard to beat what Temporal provides. Message queues feel like writing C in a world where you can just write TypeScript.