---
{"dg-publish":true,"permalink":"/post-ideas/notes-from-using-sprites-dev-for-cloud-development-environments/","created":"2026-03-23T17:12:18.713-07:00","updated":"2026-03-23T19:34:43.564-07:00"}
---

Some quick notes from building out a prototype of cloud development environments powered by Fly.io's new product: [Sprites](https://sprites.dev/). (PS: [great notes](https://simonwillison.net/2026/Jan/9/sprites-dev/#developer-sandboxes) from Simon Willison.)

We wanted isolated, ephemeral-ish environments where we can spin up an army of AI coding agents in parallel. Each with its own copy of our full product stack, its own data, and no risk of stomping on other agents' processes or storage. Sprites gives us Firecracker VMs that start fast, idle for pennies, and feel like disposable computers rather than infrastructure to manage.

Think EC2 without the headache. `sprite console` gets you a shell and we run `code tunnel` as a Sprite Service for a full IDE. The single external port limitation is a blessing in disguise: smaller attack surface, and Sprite auth sits in front of the public URL automatically. Services don't pin a Sprite as running, but foreground processes do. So you get tight control over when you're paying for compute.

While I'd like to say we're using it, I can't yet. There have been a number of road bumps during my exploration that make Sprites not yet usable for production, let alone development environments.

## What's great
* Pricing: Sprites go "warm" when not in use, meaning you only pay for storage.
* `sprite-env services`: Long-lived process management that survives session disconnects and auto-restarts on wake. 
* Pre-loaded runtimes: Node, Python, Go, Rust, Java, PostgreSQL client, git, Claude Code all come pre-installed. Our setup script went from "install everything from scratch" to "wire up our specific pieces."
* `--http-post exec`: Reliable non-interactive command execution. WebSocket-based exec had issues with connections dropping during `git clone`. For some reason, switching to HTTP POST fixed it immediately.
* Filesystem persistence: Everything written to disk survives across sessions, warm/cold cycles, and reboots. Clone once, it's there forever.
* Checkpoints: Snapshot your entire filesystem state. Useful for saving a known-good setup so you don't re-run a 10+ minute bootstrap.

## What I'm hoping they build
* No fleet/pool primitives: Creating a sprite is fast, but bootstrapping a full dev environment isn't. Would love first-class support for sprite pools or template-based creation from a checkpoint, so you can claim a pre-built environment instead of building from scratch each time.
* No background exec: `sprite exec` blocks until completion. Our setup takes 10+ minutes (npm install, build) and there's no way to fire-and-forget and check status later. Would love a `sprite exec --background` that returns a job ID.
* Higher reliability: I got a ton of random TCP timeouts and 5xx errors when listing, creating, and executing sprites. Development environments was my experiment to see how this could work for agent sandboxes in the product, but it's failing that bar by quite a bit so far.
* CLI improvements: dropping out of `sprite console` eats my ghostty session. Text formatting is all over the place. I have to `reset` to get everything back.