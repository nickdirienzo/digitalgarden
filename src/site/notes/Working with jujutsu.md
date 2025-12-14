---
{"dg-publish":true,"permalink":"/working-with-jujutsu/","created":"2025-12-13T21:56:24.003-08:00","updated":"2025-12-13T22:21:53.346-08:00"}
---

Scattered, thoughts, tips and tricks working with `jj`

I figured now would be a good time to try out `jj` since I am 5 PRs deep in a stack.

First learning: `jj` wants you to have a mental model of commit = feature. With `git`, you would think of a PR being the feature; even if the PR was a mess, you would likely squash and commit into `main` to keep it clean.

Because of this, I have to cleanup my "mess" of commits like: "yay it works", "rename", "etc". This is reasonable since I put all my effort into PR descriptions, less so individual commits.

So to take my 7 commit mess into 1 I effectively did an interactive rebase with squashing:

```bash
# This is saying from this revision set squash the changes into `wutnoxrs`
jj squash --from "wutnoxrs..yzoxlnsx" --into wutnoxrs
```

Second learning: `jj` is always operating on a commit for _everything_.  I accidentally ran the previous command as:

```bash
jj squash --from wut --into yzox
```

But this is saying take revision `wut` and teleport it into `yzox`. I thought it meant from `wut` to `yzox` create a new revision and squash them into that. Nope.

But because everything is a revision. I can easily just run `jj undo` and fix my oopsie.

Third learning: `jj` effortlessly manages stacked PRs. Let's say I realize I need to make a change to PR-A while working in PR-D. I can easily do `jj edit PR-A`, make the change, and it will auto-propagate the changes forward across the stack. Then when I push, all of those branches get pushed (effectively auto-rebased with their respective parent and pushed).

While doing my own cleanup and transition from `git` to `jj`, I found a revision in PR-D that really should be in PR-B. Now I can take my oopsie command from earlier and actually use it appropriately. I want to take revision `qplwptqv` and bring it to `wutnoxrs`; that can be done with:

```bash
jj squash --from qplwptqv --into wutnoxrs
```

And this applies that revision to the other one. This would have been so, so painful in `git` because I would have had to cherry-pick that commit into PR-B, drop it in PR-D, and then rebase PR-C and PR-D onto the latest PR-B. `jj` just did all of that for me.