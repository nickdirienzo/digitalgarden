---
{"dg-publish":true,"permalink":"/working-with-jujutsu/","created":"2025-12-13T21:56:24.003-08:00","updated":"2025-12-13T22:03:22.366-08:00"}
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
jj squash --from wut --to yzox
```

But this is saying take commit `wut` and teleport it into `yzox`. I thought it meant from `wut` to `yzox` create a new commit and squash them into that. Nope.

But because everything is a commit. I can easily just run `jj undo` and fix my oops.