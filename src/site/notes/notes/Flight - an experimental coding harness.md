---
{"dg-publish":true,"permalink":"/notes/flight-an-experimental-coding-harness/","created":"2026-04-10T14:43:02.164-07:00","updated":"2026-04-27T23:02:37.699-07:00"}
---

## Validation is the bottleneck
I've been a happy user of [Conductor](https://www.conductor.build/) for the last few months. Having automatic worktrees and a performant orchestration tool on top of Claude Code helped move me from a couple of parallel workstreams to somewhere closer to 7+ at any given moment. I tried to do something similar with Claude desktop but the app is too slow to use as a power user.

Now I'm having another problem: I'm resource constrained. I only have so much memory to give to running the Mirage stack, and running more than one at a time really slows down the rest of what I do (Slack, internet, Notion, etc). There's two camps for this problem: one that says buy a bigger machine and the other says run your environment in the cloud. 

I've been seeing more coding harnesses (Twill.ai, Coder, Zed, and I'm sure Conductor soon) try to lock you into their ecosystem and throw margin on top of rented infrastructure; rightfully so, that gives you a moat and a path to owning inference. Even Anthropic is doing this with Claude by letting you run Claude Code tasks in their infrastructure -- which if you haven't tried, is generally a great experience.

But it's still insufficient. So earlier this month, I said screw it. I don't want to wait for these tools to allow an open interface to coding in our cloud. I have AI. I can do this myself.
## Provisioning cloud workspaces
I first experimented with [Coder](https://coder.com/). This worked fairly well and gave us a pathway to spin up devboxes in our cloud using HCL. This provided some sanity around what is typically a bunch of bash. After moving to EC2 instances as a provisioning surface, I found myself writing a bunch of bash anyway -- well Claude, not me.

We have an EC2 template for Coder community edition that starts a pre-baked AMI of our development environment for marginally faster boot. This gives us the benefit of complete isolation of environments, allowing us to run the full stack without worrying about collisions on data, ports, or code. Each of these devboxes run ngrok for secure access, allowing us to share links to prototypes for quickly getting feedback.

Today, the intention is for these devboxes to be ephemeral. This reduces our patch management surface area and costs, while allowing us to spin up many EC2 instances concurrently. Already I'm seeing this step take longer than is pleasant, so we'll need to optimize here.

We had the ability to spin these up, but there was friction in getting there. Logging into Coder, clicking a button, connecting to Claude via the web app. It was all rather unpleasant, despite it feeling like a step in the right direction. I believe that good [[talks/PyBay 2023 - Infrastructure as a Product\|internal tools should be products]] in their own right, so I decided to make this experience better.
## Introducing Flight
[Flight](https://github.com/mirage-security/flight)  is an open-source, pluggable Claude Code orchestrator. It builds on a bunch of experiments in harness engineering we've run over the last few months; we don't write code by hand anymore, we iterate heavily on plans, and we like validating features end-to-end. Flight gives us a frontend to provision workspaces to run the full Mirage stack in isolation, but the lifecycle scripts are just that -- any project could use this as a frontend if they wanted to.

The initial release is a native Mac app built with Swift to build faster with Claude. It's built around a workflow of isolated worktrees, has native GitHub integrations (and a pluggable forge framework for others), and native support for remote worktree workspaces. 

Since my initial experiment, more of the team has been using and improving Flight. We're keeping it open-source so anyone can fork or contribute too. We're all shipping more code with AI, and our tools need to evolve to support high-volume code development and verification.

As long as that remote workspace supports ssh, Flight can build with Claude.
## Early observations 
The team has started to use Flight in anger this week. It's early, so more notes soon.

A few things that are exciting to see:
* The inline plan review tool changed the iteration loop more than I expected. It's a slight quality of life improvement but now I can batch review without having to copy-paste from the plan. It's like a simpler version of Ultraplan.
* Using Opus 4.6 1M means I don't really think about handoffs or compaction anymore. That used to be a real source of friction. This unlocks more native conversational programming.
* Shareable ngrok links are useful. We're giving feedback on features earlier than we ever have. It's making feature development way more collaborative and more fun because we can see live updates to the site as the agent works behind the scenes.
## Take flight
We have a bunch of ideas for where Flight goes next. Pluggability is the big one. I want this to work like VSCode extensions, not a monolith. Open source brought us to where we are today, and I'm sad to see closed-source making such a strong comeback for developer tools and infrastructure.

I would love for you to try this out, submit issues (there are many!), provide feedback, share ideas, fork it and make it your own.