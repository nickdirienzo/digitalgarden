---
{"dg-publish":true,"permalink":"/working-with-jujutsu/","created":"2025-12-13T21:56:24.003-08:00","updated":"2025-12-16T18:02:32.717-08:00"}
---

Scattered, thoughts, tips and tricks working with [`jj`](https://www.jj-vcs.dev/latest/).

I figured now would be a good time to try out `jj` since I am 5 PRs deep in a stack. As like most of everything here, this is written largely stream of consciousness as I am learning and doing.

## Commit = feature
First learning: `jj` wants you to have a mental model of commit = feature. With `git`, you would think of a PR being the feature; even if the PR was a mess, you would likely squash and commit into `main` to keep it clean.

Because of this, I have to cleanup my "mess" of commits like: "yay it works", "rename", "etc". This is reasonable since I put all my effort into PR descriptions, less so individual commits.

So to take my 7 commit mess into 1 I effectively did an interactive rebase with squashing:

```bash
# This is saying from this revision set squash the changes into `wutnoxrs`
jj squash --from "wutnoxrs..yzoxlnsx" --into wutnoxrs
```

## Everything is a revision
Second learning: `jj` is always operating on a commit for _everything_.  I accidentally ran the previous command as:

```bash
jj squash --from wut --into yzox
```

But this is saying take revision `wut` and teleport it into `yzox`. I thought it meant from `wut` to `yzox` create a new revision and squash them into that. Nope.

But because everything is a revision. I can easily just run `jj undo` and fix my oopsie.
### Even current edits
With `git`, you have to mentally differentiate between what is staged, dirty, untracked, and committed. With `jj`, you're never not operating on a revision:

```bash
@  omlsswzl nick 2025-12-13 22:23:52 66f2d3c2
│  (empty) (no description set)
```
## Stacking PRs is so much simpler
Third learning: `jj` effortlessly manages stacked PRs. Let's say I realize I need to make a change to PR-A while working in PR-D. I can easily do `jj edit PR-A`, make the change, and it will auto-propagate the changes forward across the stack. Then when I push, all of those branches get pushed (effectively auto-rebased with their respective parent and pushed).

While doing my own cleanup and transition from `git` to `jj`, I found a revision in PR-D that really should be in PR-B. Now I can take my oopsie command from earlier and actually use it appropriately. I want to take revision `qplwptqv` and bring it to `wutnoxrs`; that can be done with:

```bash
jj squash --from qplwptqv --into wutnoxrs
```

And this applies that revision to the other one. This would have been so, so painful in `git` because I would have had to cherry-pick that commit into PR-B, drop it in PR-D, and then rebase PR-C and PR-D onto the latest PR-B. `jj` just did all of that for me.

`jj` made it incredibly easy to take 40 commits across 5 stacked PRs and turn them into only 1 commit per PR and update everything all at once. I think this was like under 30 minutes of effort while learning how this new tool works (thanks AI). After I did all this squash surgery, it should be simple enough to push with:

```bash
jj git push --all
```

###  Fixing remote tracking
This was terrifying to run, so I learned there's a dry-run mode (`--dry-run`). I'm glad I did because this would push _all_ of my local bookmarks with remote... and I have _many_. Worse yet, I learned that a number of these branches are not tracked by `jj` and wouldn't be pushed because of all the changes I made:

```
Warning: Non-tracking remote bookmark nvd/feature@origin exists
Hint: Run `jj bookmark track nvd/feature@origin` to import the remote bookmark.
```

Well, I tried that and it got more complicated:

```
nickdirienzo@Nicks-MacBook-Pro mirage % jj bookmark track nvd/feature@origin
Started tracking 1 remote bookmarks.
nvd/feature (conflicted):
  + zwxzuvzt?? 818f8d8d feat(TSK-1857): some description here
  + oksymzvq ff94891d env vars
  @origin (behind by 1 commits): oksymzvq ff94891d env vars
```

Hm. Okay. I can tell `jj` to treat `zwxzuvzt` as the bookmark right?

```
nickdirienzo@Nicks-MacBook-Pro mirage % jj bookmark set nvd/feature -r zwxzuvzt
Error: Change ID `zwxzuvzt` is divergent
Hint: Use commit ID to select single revision from: a7d4633daa8d, 818f8d8d4fa1
Hint: Use `change_id(zwxzuvzt)` to select all revisions
Hint: To abandon unneeded revisions, run `jj abandon <commit_id>`
```

I can, but I can't use the change ID. I must use the underlying commit ID. But we know that from our logs. So we can run a command like this across each of the remote branches:

```bash
jj bookmark set nvd/feature -r 818f8d8d
```

After doing that for each of the bookmarks, I was able to push everything:

```bash
jj git push -b nvd/feature-A \
            -b nvd/feature-B \
            -b nvd/feature-C \
            -b nvd/feature-D \
            -b nvd/feature-E
```

I would love to run `jj git push --all`, but my dry run listed 74 bookmarks it would push. A lot of those are very, very stale. Not only is `jj` already making more productive with stacked PRs, but it is also encouraging me to Marie Kondo these all branches from my life.

## Cheatsheet
Create new revision from where we are: `jj new`

Create new revision off `main` (i.e. need to hotfix something): `jj new main`

Instead of `git commit -m`, you would describe a revision: `jj describe`

Prepare revision for PR:
* Create the bookmark: `jj bookmark create $name` ($name will be used as a branch in git)
* Push the change: `jj git push --bookmark $name` (throw on `--dry-run` if you want to double check); this makes sure it's tracked by `jj` on origin

All done with those changes? Time for another `jj new`, which locks in those changes locally under the bookmark. 

Lean into `jj new` for experiments, hotfixes, etc. Don't be afraid of having "uncommitted" changes; `jj` is managing it behind the scenes.

Looking at diffs between local and origin: `jj diff --from $bookmark$@origin`

Moving files across revisions: `jj squash -i --into <revision_id>`. This lets you select which files you want to move from the current revision into the provided revision. To commit the squash, type `c`.

Pulling `main` and rebasing revisions on top of the new changes from trunk: 

```bash
jj git fetch
# From each base revision, rebase it on top of the new changes
jj rebase -d main@origin
```

## Aliases
Here are some aliases I've been using to make my transition to `jj` a bit easier.

This shows all of the revisions from where we are to `main`:

```
[aliases]
# Show my chain of revisions on the current stack
stack = ["log", "-r", "trunk()..@"]
# Show only my local work that hasn't been pushed/synced
local-wip = ["log", "-r", "mine() & (trunk()..) ~ ::remote_bookmarks(remote='origin')"]
# "Show me all my unmerged changes"
recent = ["log", "-r", "mine() ~ ::trunk()"]
```