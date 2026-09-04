---
date: '2026-09-04T09:00:00+03:00'
draft: false
title: 'Git Command Cheatsheet'
tags: ["cheatsheet", "git"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
description: "The Git commands worth keeping close — configuration, the daily commit loop, branching and remotes, reading history, and the recovery tools you only reach for when something has gone wrong."
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
---

Most days you need about eight Git commands. The trouble is the other forty, the ones that surface when a branch is in the wrong place, a commit needs to be split, or a bug appeared somewhere in the last two hundred commits and nobody remembers when.

What follows is both halves: the daily loop first, then the recovery tools. The through-line is that Git keeps your work in three places — the working directory where you edit, the index (staging area) where you assemble the next commit, and the repository where commits become permanent. Almost every confusing command makes sense once you know which of those three it touches.

## Configuration

Two settings are mandatory, because Git stamps them onto every commit and tag you create:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

`--global` writes to your user-level config and applies everywhere. Drop the flag inside a repository to override it there — useful when work and personal commits need different addresses.

Since Git 2.28 you can pick the name new repositories use for their first branch. GitHub, GitLab and most projects moved to `main`, and setting this locally saves renaming every repo you create:

```bash
git config --global init.defaultBranch main
```

Aliases are worth the two minutes they take. The classic one is a compact history graph:

```bash
git config --global alias.glog "log --graph --oneline --decorate"
```

That makes `git glog` work anywhere. Two more settings worth knowing — the editor Git opens for commit messages and interactive rebases, and a direct route into the config file when you would rather edit it by hand:

```bash
git config --global core.editor vim
git config --global --edit
```

## Starting a repository

```bash
git init
git init my-project
```

With no argument, `git init` turns the current directory into a repository. With a name, it creates that directory first.

Cloning brings down a project along with its entire history:

```bash
git clone https://github.com/user/project.git
```

That history can be large. When you only want the current state — a build agent, a quick look at someone else's code — ask for a shallow clone:

```bash
git clone --depth=1 https://github.com/user/project.git
```

You get one commit's worth of history. `git log` will show a single entry, which is the point: the download is a fraction of the size. You can deepen it later with `git fetch --unshallow` if you change your mind.

## The daily loop

`git status` is the command to run when you are unsure of anything. It lists what is modified, what is staged, which branch you are on, and how far it has diverged from its remote.

```bash
git status
git status -s
```

The `-s` short form is one line per file and much easier to scan once you know the notation.

Staging is selective by design. You choose what goes into the next commit rather than committing everything you touched:

```bash
git add file.txt
git add src/
git add .
```

Before staging, check what you are about to include. The two diff commands answer different questions:

```bash
git diff
git diff --staged
```

`git diff` shows what you have changed but not yet staged. `git diff --staged` (also spelled `--cached`) shows what is staged and will land in the next commit. Reviewing the second one before every commit catches an astonishing number of stray debug lines.

Then commit:

```bash
git commit -m "Fix header alignment on mobile"
```

Without `-m` Git opens your editor, which is the better option for anything that deserves a body paragraph explaining *why*.

To stop tracking a file and delete it in one step:

```bash
git rm file.txt
git rm --cached file.txt
```

`--cached` removes it from Git but leaves it on disk — the fix for a config file or a secret you committed by accident and now want ignored.

## Undoing work before it is committed

Modern Git splits this job between two commands. `git restore` deals with file contents; `git switch` deals with branches. Both were introduced in Git 2.23 to replace the overloaded `git checkout`, and both are stable — `checkout` still works, but the newer commands are much harder to misuse.

To throw away uncommitted edits to a file:

```bash
git restore file.txt
```

This is unrecoverable. The changes were never committed, so Git has no copy of them.

To unstage something without touching your edits:

```bash
git restore --staged file.txt
```

The older spellings of these two are `git checkout -- file.txt` and `git reset file.txt` respectively, which you will still see in scripts and older documentation.

Neither command touches untracked files. Those need `git clean`, and this is a command to always dry-run first:

```bash
git clean -nd
git clean -df
```

`-n` shows what would be deleted, `-d` includes directories, `-f` actually does it. Running `-df` without checking `-nd` first is a reliable way to lose an hour of work that was never in Git at all.

## Stashing

When you need to switch context but the work is not ready to commit:

```bash
git stash
git stash pop
```

`git stash` shelves your uncommitted changes and gives you a clean working directory. `git stash pop` puts them back and drops the stash entry. Stashes form a stack, so you can list them and discard ones you no longer need:

```bash
git stash list
git stash drop stash@{1}
```

## Branching

```bash
git branch
git branch -a
```

Plain `git branch` lists local branches; `-a` adds the remote-tracking ones. Add `-v` to see each branch's latest commit and how it stands relative to its upstream.

Creating and switching:

```bash
git switch -c feature/login
git switch main
```

`-c` creates the branch and moves to it. Without it you switch to a branch that already exists. The `checkout` equivalents are `git checkout -b feature/login` and `git checkout main`.

Merging brings another branch into the one you are on:

```bash
git switch main
git merge feature/login
```

Deleting is deliberately awkward when it might lose work:

```bash
git branch -d feature/login
git branch -D feature/login
```

`-d` refuses if the branch has commits that are not merged anywhere. `-D` forces it. Use `-d` by default and let it stop you.

After a long project, local branches accumulate. To clear out everything except `main`:

```bash
git branch | grep -v "main" | xargs git branch -D
```

That is a force delete on everything it matches, so run the `git branch | grep -v "main"` part on its own first and read the list.

Renaming a branch — including the `master` to `main` rename on an older project — is `-m`:

```bash
git branch -m master main
```

If the branch also exists on a remote, rename it in the hosting provider's interface as well, then repoint your local branch with `git push -u origin main`.

## Remotes

A remote is a named URL. `origin` is the conventional name for the one you cloned from, but nothing is special about it:

```bash
git remote -v
git remote add upstream https://github.com/original/project.git
git remote get-url origin
```

Git is distributed, so you can have as many remotes as you like and push to whichever you want:

```bash
git remote add working https://gitlab.com/user/mirror.git
git push working main
```

Fetching downloads new commits without changing your working directory:

```bash
git fetch origin
git fetch --prune origin
```

`--prune` deletes local references to remote branches that no longer exist upstream — worth adding whenever your branch list feels stale.

`git pull` is a fetch followed immediately by a merge into your current branch:

```bash
git pull origin main
git pull --rebase origin main
```

`--rebase` replays your local commits on top of what you fetched instead of creating a merge commit, which keeps the history linear.

Pushing sends commits the other way:

```bash
git push origin main
git push -u origin feature/login
git push --tags origin
```

`-u` sets the upstream, so subsequent `git push` and `git pull` on that branch need no arguments.

Force pushing rewrites the remote branch and can destroy someone else's commits. When you genuinely need it — usually after an interactive rebase on a branch only you use — prefer the safer form:

```bash
git push --force-with-lease origin feature/login
```

`--force-with-lease` refuses if the remote has commits you have not seen, which is exactly the case where plain `--force` would silently delete a colleague's work.

## Keeping a fork in sync

A fork does not update itself. When the original project moves on, your fork stays where it was. The fix is a second remote, conventionally named `upstream`, pointing at the project you forked from:

```bash
git remote add upstream https://github.com/original/project.git
git remote -v
```

You should now see `origin` (your fork) and `upstream` (the original), each listed for fetch and push. Bringing your fork up to date is then a fetch and a merge:

```bash
git switch main
git fetch upstream
git merge upstream/main
git push origin main
```

Do this before starting any new branch of work. Rebasing a week-old feature branch onto a moved target is considerably less pleasant than starting from a current one.

## Reading history

`git log` in its default form is verbose. These four are the ones worth muscle memory:

```bash
git log --oneline
git log -5
git log --oneline --graph --decorate
git log -p
```

One commit per line; the last five commits; a text graph with branch and tag labels attached; and the full diff of each commit.

Filtering is where log becomes a search tool rather than a wall of text:

| Command | What it finds |
| --- | --- |
| `git log --oneline --after="24 months ago"` | Commits since a date, in plain English or ISO form |
| `git log --oneline --before="2026-01-01"` | The other end of a range |
| `git log --grep="[Aa]nimation"` | Commits whose message matches a pattern |
| `git log --author="Maria"` | Commits by one person |
| `git log -- src/index.html` | Commits that touched one file |
| `git log --stat` | Which files changed, with line counts |
| `git log main..feature` | Commits on `feature` not yet in `main` |

These combine, which is where the real value is. A concise history of animation work from the last two years is one command:

```bash
git log --oneline --after="24 months ago" --grep="[Aa]nimation"
```

To see a single commit in full, including its diff:

```bash
git show a1b2c3d
```

And to find out who last changed each line of a file, and in which commit:

```bash
git blame build/index.html
git blame -s build/index.html
```

`-s` suppresses the author name and timestamp, leaving just the commit hashes — a tighter view when you only want to jump to the commits.

## Tags

Tags mark a commit as significant, usually a release:

```bash
git tag
git tag v2.0.1
git tag -a v2.0.1 -m "Release 2.0.1"
git tag -d v2.0.1
```

Plain `git tag <name>` creates a lightweight tag, a bare pointer to a commit. `-a` creates an annotated tag, a real object with an author, date and message. Use annotated tags for anything you publish; lightweight ones are fine as private bookmarks.

Add a commit hash to tag something other than `HEAD`:

```bash
git tag v1.0.0 a1b2c3d
```

Tags are not pushed with a normal `git push`. Send them explicitly with `git push --tags origin`, as noted above.

## Rewriting history

Everything in this section changes commits that already exist. That is safe on commits you have not shared, and disruptive on commits others have pulled.

To fix the commit you just made:

```bash
git commit --amend -m "A better message for the previous commit"
git commit --amend --no-edit
```

The first replaces the last commit with the staged changes plus a new message. The second folds staged changes into the last commit while leaving its message alone — the standard move when you forgot a file.

Interactive rebase is the tool for tidying a run of commits before opening a pull request:

```bash
git rebase -i HEAD~4
```

That opens the last four commits in your editor, one per line, each prefixed with a command. Change `pick` to `squash` to fold a commit into the one above it, `reword` to edit its message, `drop` to delete it, or reorder the lines to reorder the commits. It is how a branch containing "fix typo", "fix typo again" and "actually fix it" becomes a single coherent commit.

A plain rebase moves your branch onto a new base:

```bash
git rebase main
```

## Moving specific commits between branches

When you want two or three commits from another branch rather than the whole thing, cherry-pick them:

```bash
git switch main
git cherry-pick a1b2c3d e4f5g6h
```

Each one is applied to your current branch as a new commit. Adding `-n` applies the changes to your working directory and index without committing, so you can review or combine them before making a single commit:

```bash
git cherry-pick a1b2c3d e4f5g6h -n
```

Get the hashes from `git log --oneline` on the source branch first.

## Undoing committed work

Two commands do this, and the difference matters.

`git revert` creates a *new* commit that undoes an old one:

```bash
git revert a1b2c3d
```

Nothing is erased, so this is the safe choice for anything already pushed.

`git reset` moves the branch pointer backwards, and its three modes differ in how much they take with them:

```bash
git reset --soft HEAD~1
git reset HEAD~1
git reset --hard HEAD~1
```

`--soft` moves the branch only, leaving your changes staged. The default `--mixed` also resets the index, leaving changes in your working directory as unstaged edits. `--hard` resets the working directory too and discards everything after that commit.

### Recovering from a bad reset

`git reset --hard` looks final, and mostly it is not. Git keeps a log of everywhere `HEAD` has pointed, including the positions you reset away from:

```bash
git reflog
```

Each line has a hash and a description of what moved `HEAD` there. Find the state you want back and reset to it:

```bash
git reset --hard a1b2c3d
```

The reflog is local, unshared, and expires after ninety days by default. It has rescued more work than any other Git command, and it is worth checking before concluding that something is gone.

## Finding the commit that broke something

When a bug exists now and did not exist at some earlier point, `git bisect` finds the commit responsible by binary search. Twenty commits take about four tests; a thousand take about ten.

```bash
git bisect start
git bisect bad
git bisect good a1b2c3d
```

You mark the current state as bad and a known-good commit from `git log --oneline`. Git checks out a commit halfway between them. Test it, then tell Git what you found:

```bash
git bisect good
git bisect bad
```

Repeat until Git names the first bad commit. Then end the session and return to where you started:

```bash
git bisect reset
```

Forgetting `git bisect reset` leaves you on a detached checkout in the middle of history, which is a confusing state to walk away from.

If the bug can be detected by a script or a test, hand the whole thing over:

```bash
git bisect run npm test
```

Git runs the command at each step and uses its exit code — zero for good, non-zero for bad — to drive the search unattended.

## Working on two branches at once

Stashing and switching gets tedious when you need two branches side by side — comparing behaviour, or fixing a bug while a long build runs. A worktree gives you a second checked-out directory backed by the same repository:

```bash
git worktree add ../project-hotfix
git worktree list
cd ../project-hotfix
```

`git worktree add` creates the directory and checks out a branch there. Both directories share one `.git`, so commits made in either are immediately visible to the other, and there is no second clone to keep in sync. Clean up when you are done:

```bash
git worktree remove ../project-hotfix
```

## Exporting a snapshot

`git archive` packages a commit's tree as an archive, with no `.git` directory and no history:

```bash
git archive -o ../release.zip main
git archive -o ../build.zip feature/login -- build
```

The second form limits the archive to one path. This is the right way to hand someone a copy of the code without the repository, and it beats copying the folder by hand — the contents come from Git, so ignored and untracked files are never included.

## Ignoring files

A `.gitignore` file at the root of the repository keeps generated and local files out of `git status` and out of commits:

```text
/logs/*
!logs/.gitkeep
/tmp
*.swp
.env
```

Patterns are relative to the file's own directory and apply to subdirectories too. A leading `!` re-includes something an earlier pattern excluded — above, the `logs/` directory stays in the repository via `.gitkeep` while its contents are ignored.

`.gitignore` only affects untracked files. A file Git is already tracking keeps being tracked no matter what you add to the list; use `git rm --cached` to stop that, as described earlier.

## Where to look next

`git help <command>` opens the full manual for anything here, and the official reference at [git-scm.com/docs](https://git-scm.com/docs) is genuinely good. For the underlying model rather than the commands, *Pro Git* by Scott Chacon and Ben Straub is free online at [git-scm.com/book](https://git-scm.com/book) and remains the best explanation of why Git works the way it does.
