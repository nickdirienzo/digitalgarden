---
{"dg-publish":true,"permalink":"/learnings-from-building-inc/","created":"2026-01-25T12:18:29.562-08:00","updated":"2026-01-26T22:48:56.057-08:00"}
---

I'm experimenting with agent orchestration on top of Claude Code. It's inspired a lot by gastown and others, but it's my own personal take on it. It's called [inc](https://github.com/nickdirienzo/inc).

More to write about this, but the short thesis is: because agents don't have human ramp up costs, we can break free of Conway's Law -- the "org structure" (ephemeral virtual teams) mirrors the problem structure, instead of the architecture mirroring the org structure.

* Restricting access to files is kind-of hard today with Claude Code within a spawned directory. Yes you can restrict Edit/Write, but if the agent has Bash, all bets are off. Fortunately, it seems like it will only have access to files within its spawned directory, but there's a need for improved filesystem sandboxing.
* I'm not sure if a TUI is the way, but it's kinda neat. 
* Agents are still really bad at complex instruction following, so guardrails are a must. Otherwise your repo will get rekt:
	* Case 1: Coder agents really like trying to read the default workspace files even though they operate in a completely separate codebase![Pasted image 20260125161139.png](/img/user/Pasted%20image%2020260125161139.png)
	* Case 2: the PM agent completing a spec and spinning anyway ![Pasted image 20260125201656.png](/img/user/Pasted%20image%2020260125201656.png)
* Claude Skills are awesome. Most of this is powered by Claude Skills and a thin CLI tool to manage file writing synchronization.
* The technologies we've used as humans to communicate: files, emails, etc are still valid for agents. Right now, inc is using files to request attention, but I think the next iteration of this is to use email or some task management system. It kinda could be _anything_ that lets people communicate. 