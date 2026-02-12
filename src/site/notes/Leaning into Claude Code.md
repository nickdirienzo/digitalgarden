---
{"dg-publish":true,"permalink":"/leaning-into-claude-code/","created":"2026-01-27T12:35:28.878-08:00","updated":"2026-01-27T13:05:40.348-08:00"}
---

I've had a couple of friends ask how I use Claude Code, so figured to write it down here.

---

At least at the moment, I view it as a skill on its own. Being able to stay on top of 7+ concurrent workstreams that are pretty different while still maintaining taste is tough. The current tooling doesn't make it much easier.

When I first started using Claude Code more seriously (July 2025), I was mostly just using Claude Code one at a time. I was tinking about features in smaller incremental blocks instead of a larger milestone. When I first started, I was still in the prior mindset, so I would be pretty hands-on, I wasn't really doing planning, it was a lot of prompting but in the way I would delegate work to an engineer. 

Over the next few months, I started to get a little bit more ambitious. I introduced a self-hosted Coder instance where I could start having CC work in the background mostly autonomously. What really unlocked parallelization for me was Claude Desktop introducing Code. It has built-in support for worktrees, which made the need for Coder a bit redundant (we still have it for now). 

Two things shifted in my head around the same time, starting to use [[Working with jujutsu \| jj]] and seeing [Boris' workflow on X](https://x.com/bcherny/status/2007179832300581177):![Pasted image 20260127124351.png](/img/user/Pasted%20image%2020260127124351.png)

The other change in workflow I made was Plan Mode. I articulated the product vision and the high-level architecture, and then let Claude interpret my scattered thoughts into a concrete plan it could execute on. I would review the plan as a file and send suggestions as chat, which would then update the file. Once the plan looked good, I let it auto-edit until it's ready for a PR. 

There's some foundational investments that make this safe to do for us. Early on I laid the groundwork for ADRs which provides historical context on why we've made certain significant architectural decisions, we have a giant TypeScript monorepo so everything is grokkable in one directory, and we have way more test coverage than I think a seed stage company should have. The cost for code is 0 so might as well have tests.

With this workflow, I started to see a natural progression from 3 -> 5 -> 7 CC tasks in parallel thanks for Claude Desktop Code having native worktree support. Sprinkled with a Coder task here and there. This enabled me to move up from code as the artifact I was reviewing to instead reviewing plans, which is where I like operating. 

For fun, I tried to not open my editor for a week and was largely successful. I would review the plans in chat -- which Claude Desktop makes it less painful visually. I would have it put up a PR, Claude Code would review the PR, I would then have Claude Code iterate on the changes. I would do a pass on the change and leave comments on the PR, which then I would tell CC to review and fix. This worked. 

With this level of work in parallel, I find myself bombarded with notifications. I haven't fully adopted full YOLO mode because I've seen CC regress into using `sed` instead of `Edit`/`Write` so it hasn't earned that level of trust yet. BUT, I think that's solvable.

I think there's a more autonomous workflow here that I'm [tinkering with](https://github.com/nickdirienzo/inc), but just moving up to the high-level alone -- even with all the road bumps in the current iteration of UX -- has enabled me to effectively double my output.

---
## Relevant links
* https://x.com/bcherny/status/2007179832300581177
* https://www.geoffreychallen.com/talks/2026-01-15-a-day-with-claude-using-and-teaching-coding-agents
* https://x.com/karpathy/status/2015883857489522876