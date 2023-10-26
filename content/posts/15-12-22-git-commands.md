---
title: "Git Commands (reference)"
description: "Specific git command reference"
date: 2022-12-15
lastmod: 2023-10-26
draft: false
slug: git-commands
weight: 5
---

Note: See [previous](/posts/clear-overview-git) post

```sh
git fetch
```
```sh
git pull
```
```sh
git push
```
```sh
git cherry-pick <commit>
```

<br>

```sh
git rebase [dest-branch]
```
```sh
git merge [dest-branch]
```
rebase or merge current branch or commit (the one head is pointing to) with [dest-branch]

<br>

```sh
git status
```

- Will, among other things, show the status of a merge in progress.
-  During a merge, short Format "-s" is a useful option, will show (for each path):
    - A completed merge or no merge: X: Index, Y: Working directory
    - A merge in progress: XY are changes introduced by each merge candidate (note: unmodified files, aka "paths", are not listed)

<br>

```sh
git revert <commit>
```
Unlike reset will create a new commit where the everything is the same except for the specified commit (i.e. doesn't include history)

```sh
git restore .
```

Restores unstaged files in the working directory from the index.

<br>

```sh
git reset --option <target>
```
- From [the manual](https://git-scm.com/book/en/v2/Git-Tools-Advanced-Merging#_undoing_merges), reset --hard usually goes through three steps:
    1. Move the branch HEAD points to.
    2. Make the [index](/posts/clear-overview-git/#the-three-trees) look like HEAD.
    3. Make the [working directory](/posts/clear-overview-git/#the-three-trees) look like the index.
- Like git checkout except it moves the HEAD *as well as* the current branch pointer—leaving the node(s) orphaned
- "git reset" defaults to "git reset --mixed HEAD"
- Will reset back to specified commit (or where HEAD is pointed to), removing commits inbetween as well.

```sh
git rev-parse --show-toplevel
```
From [this](https://stackoverflow.com/a/12293994/2083958) stackoverflow post. Will show the location of the git repo when in a subdirectory.