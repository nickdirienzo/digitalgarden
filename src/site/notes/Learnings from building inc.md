---
{"dg-publish":true,"permalink":"/learnings-from-building-inc/","created":"2026-01-25T12:18:29.562-08:00","updated":"2026-01-26T21:58:42.752-08:00"}
---

* Restricting access to files is kind-of hard today with Claude Code within a spawned directory. Yes you can restrict Edit/Write, but if the agent has Bash, all bets are off. Fortunately, it seems like it will only have access to files within its spawned directory, but there's a need for improved filesystem sandboxing.
* I'm not sure if a TUI is the way, but it's kinda neat. 
* Agents are still really bad at complex instruction following, so guardrails are a must. Otherwise your repo will get rekt:
	* Case 1: Coder agents really like trying to read the default workspace files even though they operate in a completely separate codebase![Pasted image 20260125161139.png](/img/user/Pasted%20image%2020260125161139.png)
	* Case 2: the PM agent completing a spec and spinning anyway ![Pasted image 20260125201656.png](/img/user/Pasted%20image%2020260125201656.png)
* Claude Skills are awesome. Most of this is powered by Claude Skills and a thin CLI tool to manage file writing synchronization.