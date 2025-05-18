---
{"dg-publish":true,"permalink":"/obsidian-digital-garden-on-cloudflare/"}
---

I'm using the [Obsidian Digital Garden Plugin](https://dg-docs.ole.dev) for my [[Digital Gardening\|Digital Gardening]].

The plugin's default recommendation is Vercel. I tend to not use Vercel for a variety of reasons.

Instead I'm publishing on Cloudflare Pages. I've used CF Pages for a number of static sites and single page apps over the last couple of years. It's awesome to be able to connect a GitHub repo to Cloudflare and have them manage CICD. It's been a great experience and it'll be fun to use more of the Cloudflare products over time. If I start to lean into Cloudflare more for personal infrastructure, I'll need to figure out their Terraform provider. For now ClickOps is the way.

Sadly, it didn't immediately work when selecting 11ty as the framework. It's because there's some custom build tooling. I had to set the following options and then everything was good:
- Framework preset: None
- Build command: `npm run build`
- Build output directory: `dist`

As a fun bonus of this side quest, Cloudflare now manages my DNS.