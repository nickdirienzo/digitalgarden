---
{"dg-publish":true,"permalink":"/til/git-hub-pr-comments-live-in-three-different-places/","tags":["til"],"created":"2026-08-13T23:36:34.695-07:00","updated":"2026-08-14T09:27:57.707-07:00","dg-note-properties":{"tags":["til"]}}
---

TIL that `gh pr view --json comments` does not return inline review comments, which Copilot makes heavy use of.

I found this because one of my coding agents confidently declared a PR clean while Copilot's findings (including a real bug) were left unacknowledged. The agent ran the obvious command, saw nothing, and moved on.

GitHub PRs have three separate ways of accessing comments:
- Issue comments (the main conversation thread): `gh pr view --json comments`
- Review bodies (the summary attached to an approval/request-changes): `gh pr view --json reviews`
- Inline review comments (the ones anchored to actual diff lines): `gh api repos/{owner}/{repo}/pulls/{n}/comments`

There is no `gh pr view` field for the third one. If a reviewer leaves only inline comments, `gh pr view --json comments` can return an empty `comments` array even though the PR has actionable review feedback.

And GitHub has managed to make this even more confusing by having Copilot post its review summary under the login `copilot-pull-request-reviewer[bot]` but its inline findings under Copilot. So even if you know about the third API and filter by the reviewer you're looking for, you get an empty list.

We've since added a line to our `AGENTS.md` telling agents to check all three channels. 

Hope this helps someone else.