---
{"dg-publish":true,"permalink":"/obsidian-digital-garden-on-cloudflare/"}
---

I'm using the [Obsidian Digital Garden Plugin](https://dg-docs.ole.dev) for my [[musings/Digital Gardening\|Digital Gardening]].

The plugin's default recommendation is Vercel. I tend to not use Vercel for a variety of reasons.

Instead I'm publishing on Cloudflare, but it didn't immediately work. 

So here's some notes in case it's helpful to future me or someone else.

You'll want to set the following settings on Cloudflare Pages:

- Framework preset: None
- Build command: npm run build
- Build output directory: dist