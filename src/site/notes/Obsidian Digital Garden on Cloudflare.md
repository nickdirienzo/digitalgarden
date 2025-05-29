---
{"dg-publish":true,"permalink":"/obsidian-digital-garden-on-cloudflare/","created":"2025-05-17T13:40:31.061-07:00","updated":"2025-05-17T19:15:58.398-07:00"}
---

I'm using the [Obsidian Digital Garden Plugin](https://dg-docs.ole.dev) for my [[Digital Gardening\|Digital Gardening]]. While I haven't used Obsidian Publish, it's reduced my barrier to publishing on the internet. Write some thoughts down, tag it as `dg-publish: true`, `cmd+p` Publish, done.

The plugin's default recommendation is Vercel. I tend to not use Vercel for a variety of reasons.

Instead I'm publishing on Cloudflare Pages. I've used Cloudflare Pages for a number of static sites and single page apps over the last couple of years. It's awesome to be able to connect a GitHub repo to Cloudflare and have them manage CICD. 

It'll be fun to use more of the Cloudflare products over time. If I start to lean into Cloudflare more for personal infrastructure, I'll need to figure out their Terraform provider. For now ClickOps is the way.

Anyway, after connecting the repo, it didn't immediately work when selecting 11ty as the framework. It's because there's some custom build tooling. I had to set the following options and then everything was good:
- Framework preset: None
- Build command: `npm run build`
- Build output directory: `dist`

As a fun bonus of this side quest, Cloudflare now manages my DNS.