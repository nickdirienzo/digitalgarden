---
{"dg-publish":true,"permalink":"/slack-notifications-via-ntfy-sh/","created":"2026-02-15T13:52:34.973-08:00","updated":"2026-02-15T14:02:41.839-08:00"}
---

I've been trying to not be too attached to my phone. Over the last couple of years, I've dropped Tiktok, Instagram, YouTube Shorts (clearly, there's a pattern). Every time I have Slack on it, I end up treating it like an infinite feed too, so I try not to have it on my phone either.

What's challenging is I'm also on-call for everything. While we have PagerDuty setup for automated incident creation via signals, not everything is covered. So when I get @-mentioned, I'd like to know right away.

Fortunately, it looks like a wonderful open-source service exists to help do just that: https://ntfy.sh/. Between that and Claude Code, I now have an 80 line script to do exactly what I need: when I'm @-mentioned in our alerts channel, I am notified on my phone. I rarely go anywhere without my laptop, so it's really easy to jump right on as needed.

Anyway it's 80 lines long. You can find [the gist here](https://gist.github.com/nickdirienzo/66e2fe673d4fd662e609fe3984741778), and run it wherever your heart desires.