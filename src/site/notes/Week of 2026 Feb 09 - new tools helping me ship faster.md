---
{"dg-publish":true,"permalink":"/week-of-2026-feb-09-new-tools-helping-me-ship-faster/","created":"2026-02-11T20:24:21.136-08:00","updated":"2026-02-11T20:31:54.414-08:00"}
---

I feel like I hit a new level of productivity this week due to some changes in my tooling.

**conductor.build**

I wanted a better frontend for Claude Code (partially why I was building [inc](https://github.com/nickdirienzo/inc), but that's more of a software factory experiment). I found that this week in [Conductor](https://www.conductor.build/). I shared with multiple friends after an hour of use: "it feels like I'm flying." 

Today I had about 10 workspaces open, each with multiple tabs; it's a native Mac app so it doesn't suffer from the same performance problems that I get with Claude Desktop -- but with my usage, I've found it too sometimes be sluggish.

**Merge Queues**

I've been meaning to set these up for months. Auto-merging is such an immediate payoff. And Claude Code makes it a breeze to migrate our apparently 46 GHA jobs over to use merge queues. Our Conductor usage in the last 48 hours has made it an obvious need.

