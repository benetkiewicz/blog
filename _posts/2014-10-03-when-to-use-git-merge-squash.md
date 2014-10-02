---
layout: post
title:  "When to use git merge --squash"
date:   2014-10-02 16:57:59
categories: git
---

One of the greatest things in git is that a skilled developer can maintain a history that actually _mean_ something. Clean, self-contained, single-responsibility commits really help analyze code and search for bugs and regressions.

One of the toys in git toy basked that helps maintain order is `git merge SomeBranch --squash` command.

###What does squash do?
When you merge some branch to the main branch with a `--squash` option, git will produce a single set of changes, essentially flattening history of the source branch.

###What's it good for?
This is great way to merge branches where you experimented a lot, produced a lot of commits and have meaningless commit messages: _initial commit_, _binding works now_, _temporary hack for regex_, etc. I use it mainly for two things:

* prototyping or hacking - when unexpectedly something good comes out of it and I want to include that value in the main branch
* merging _the next day_ branches. Sometimes small features do not deserve their own feature branches but implementation takes longer then I initially thought. I always push my code to repository before I leave the office but it is not always good enough for everyone (inluding Continuous Integration process) to see it. On the next day I finish the task, merge with squash and remove the temporary branch.

###When to avoid squash?
Whenever there's a possibility you will want to come back to the branch after merging and add/change code, `--squash` is probably not your thing. So that would include:

* feature branches
* release branches
* branches extensively used by more than one developer
* any remote branches that are meant to stay remote for long

###Example scenarios where squash will brake things

####Fixing the feature branch

1. Implement stuff in feature branch
2. Merge that branch with squash to main branch
3. Realize there's a bug in feature branch and fix it
4. Merge again - will create conflicts (__potential fix:__ `git cherry-pick`)

####Rebasing against main branch

1. Implement stuff in feature branch
2. Merge that branch with squash to main branch
3. Changes in the main branch are coming
4. Rebase will cause conflicts
