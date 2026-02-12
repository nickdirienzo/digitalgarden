---
{"dg-publish":true,"permalink":"/shipping-faster-with-conductor-build-and-merge-queues/","created":"2026-02-11T20:24:21.136-08:00","updated":"2026-02-11T20:44:31.094-08:00"}
---

I feel like I hit a new level of productivity this week due to some changes in my tooling.

**conductor.build**

I wanted a better frontend for Claude Code (partially why I was building [inc](https://github.com/nickdirienzo/inc), but that's more of a software factory experiment). I found that this week in [Conductor](https://www.conductor.build/). Within a day I was already telling a ton of friends about it:

![Pasted image 20260211203710.png](/img/user/Pasted%20image%2020260211203710.png)

Today I had about 10 workspaces open, each with multiple tabs; it's a native Mac app so it doesn't suffer from the same performance problems that I get with Claude Desktop -- but with my usage, I've found it too sometimes be sluggish.

While I've largely stopped using VS Code, I still go to GitHub for PR review, status check resolution, and releases. It's very possible that Conductor could be my entire command center for shipping; they already have built-in check failure resolution and merging, so I fully expect them to continue to obviate the need to go anywhere else. Excited to see where they take it.

**Merge Queues**

I've been meaning to set these up for months as we've been shipping at an increasing rate with Claude Code, but our Conductor usage in the last 48 hours has made it an obvious need.

Auto-merging is such an immediate payoff. Claude Code makes it a breeze to migrate our apparently 46 GHA jobs over to use merge queues. 

I do find it very ridiculous that this is behind the Enterprise tier on GitHub. Yes, I could explore Graphite, GitLab, and any other alternative, but I don't see a need in shifting the platform we use... yet. With GitHub's continuing outages though, I won't say it hasn't crossed my mind.
