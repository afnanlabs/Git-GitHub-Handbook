# 04 — Branching and Merging

> Branches are the core of Git workflow. This file covers the full branch lifecycle, merge strategies, rebase, conflict resolution, and branch naming conventions.

---

## Table of Contents

1. [What a Branch Is](#what-a-branch-is)
2. [Creating and Switching Branches](#creating-and-switching-branches)
3. [Branch Naming Conventions](#branch-naming-conventions)
4. [Merging — Overview](#merging-overview)
5. [Fast-Forward Merge](#fast-forward-merge)
6. [Three-Way Merge (Merge Commit)](#three-way-merge)
7. [Squash Merge](#squash-merge)
8. [Merge Conflicts](#merge-conflicts)
9. [Rebasing — Deep Explanation](#rebasing)
10. [Interactive Rebase](#interactive-rebase)
11. [Detached HEAD State](#detached-head-state)
12. [Feature Branch Workflow](#feature-branch-workflow)
13. [Branch Lifecycle — Full Example](#branch-lifecycle)
14. [Beginner Mistakes](#beginner-mistakes)

---

## What a Branch Is

A branch is a **lightweight movable pointer to a commit**.

When you create a branch, Git creates a new file in `.git/refs/heads/` containing the SHA of the commit it points to. That's literally all a branch is — a 41-byte text file.

```
.git/refs/heads/main           → a3f5c1d2...
.git/refs/heads/feature-login  → a3f5c1d2...  (same commit at creation)
```

When you make a new commit on `feature-login`, only that pointer advances:

```
Before commit:
  main          feature-login
    ↓               ↓
A ← B ← C ← D

After commit on feature-login:
  main      feature-login
    ↓            ↓
A ← B ← C ← D ← E
```

`main` hasn't moved. The two branches now diverge.

---

## Creating and Switching Branches

```bash
# Create (don't switch)
git branch feature-login

# Create and switch
git switch -c feature-login           # Modern
git checkout -b feature-login         # Classic

# Switch to existing branch
git switch main
git checkout main

# Switch to previous branch
git switch -
git checkout -

# Create branch from a specific commit or branch
git switch -c hotfix/auth-bug main    # Branch from main
git switch -c experiment a3f5c1d      # Branch from a commit hash
```

### Listing branches
```bash
git branch          # Local branches (* = current)
git branch -a       # All (local + remote)
git branch -v       # With last commit hash and message
git branch --merged # Already merged into current branch (safe to delete)
```

---

## Branch Naming Conventions

Good naming is a professional habit. Teams rely on branch names for automation (CI/CD triggers, auto-labeling, PR routing).

### Common prefixes

| Prefix | Purpose | Example |
|---|---|---|
| `feature/` | New functionality | `feature/user-authentication` |
| `fix/` or `bugfix/` | Bug fix | `fix/null-pointer-login` |
| `hotfix/` | Urgent prod fix | `hotfix/payment-down` |
| `chore/` | Maintenance, deps | `chore/upgrade-dependencies` |
| `docs/` | Documentation | `docs/api-reference-update` |
| `refactor/` | Code restructure | `refactor/extract-auth-service` |
| `test/` | Test work | `test/add-e2e-coverage` |
| `release/` | Release branch | `release/v2.1.0` |
| `experiment/` | Throwaway spikes | `experiment/graphql-migration` |

### Rules
- Use **lowercase** and **hyphens** (not underscores or camelCase)
- Be **descriptive** but not excessively long: `fix/login-redirect` not `fix/fixed-the-redirect-issue-on-login-page`
- Include **issue number** when relevant: `feature/142-user-profile`
- Avoid: spaces, special characters, uppercase

---

## Merging Overview

Merging integrates changes from one branch into another. Git has several merge strategies, each with different history implications.

**Always merge into the target branch:**
```bash
git switch main
git merge feature-login   # Brings feature-login into main
```

---

## Fast-Forward Merge

Happens when the target branch hasn't moved since the feature branch was created — there's no divergence, just a straight line.

```
Before:
  main
    ↓
A ← B ← C
             ↑
         feature-login
             ↓
         C ← D ← E

After fast-forward merge:
              main = feature-login
                    ↓
A ← B ← C ← D ← E
```

Git just moves the `main` pointer forward to E. **No merge commit is created.**

```bash
git switch main
git merge feature-login    # Fast-forward happens automatically if possible
```

### Pros and cons

| Pros | Cons |
|---|---|
| Clean linear history | You lose the record that feature was on a separate branch |
| Easy to read | Can be confusing when auditing what was part of a release |

### Prevent fast-forward (always create merge commit)
```bash
git merge --no-ff feature-login
```
This creates a merge commit even when fast-forward is possible. Teams that want to track feature boundaries use `--no-ff`.

---

## Three-Way Merge

Happens when both branches have diverged — each has commits the other doesn't.

```
Before:
  main
    ↓
A ← B ← C ← F
          ↑
          └── D ← E  (feature-login)

After three-way merge:
  main
    ↓
A ← B ← C ← F ← M   (M = merge commit)
               ↑
          D ← E

M has two parents: F (from main) and E (from feature)
```

```bash
git switch main
git merge feature-login   # Creates merge commit M
```

Git uses the **common ancestor** (C) plus the two tips (F and E) to compute the merge — that's the "three" in three-way.

### Merge commit message
Default: `Merge branch 'feature-login' into main`  
You can customize: `git merge feature-login -m "merge: integrate user authentication"`

---

## Squash Merge

Takes all commits from a feature branch and collapses them into a single staged change, which you then commit manually.

```
Feature branch history:
D: "wip"
E: "fix typo"
F: "actually fix the bug"
G: "add tests"

After squash merge into main:
A ← B ← C ← S  (S = one commit with all of D+E+F+G combined)
```

```bash
git switch main
git merge --squash feature-login   # Stages all changes, doesn't commit
git commit -m "feat: add user authentication (#42)"   # You write the clean commit
```

### When to use squash merge
- Feature branches with messy WIP commits
- You want main to have one clean commit per feature
- Common in GitHub "Squash and merge" PR strategy

### Downside
The individual commits from the feature branch are **not in main's history**. The branch must be deleted afterward (it won't show as merged via `--merged` check).

---

## Merge Conflicts

Conflicts occur when the same lines of the same file were changed differently in both branches being merged.

### What a conflict looks like in a file

```
<<<<<<< HEAD (your current branch — main)
const timeout = 5000;
=======
const timeout = 3000;
>>>>>>> feature-login (the branch being merged in)
```

- `<<<<<<< HEAD` to `=======`: your version
- `=======` to `>>>>>>>`: incoming version

### Conflict resolution workflow

```bash
# Step 1: Attempt the merge
git merge feature-login

# Output: CONFLICT (content): Merge conflict in src/config.js

# Step 2: Check what's conflicted
git status
# both modified: src/config.js

# Step 3: Open the file, manually resolve
# Edit the file to the correct final state, remove all conflict markers

# Step 4: Stage the resolved file
git add src/config.js

# Step 5: Complete the merge
git commit
# (Git pre-populates the merge commit message)
```

### Tools for conflict resolution

```bash
git mergetool              # Opens configured merge tool
git config --global merge.tool vscode
# VS Code has built-in conflict UI: "Accept Current" / "Accept Incoming" / "Accept Both"
```

### Aborting a merge
```bash
git merge --abort    # Return to pre-merge state
```

### Tips for avoiding conflicts
- Keep branches short-lived (merge often)
- Communicate with teammates about what files you're editing
- Pull/rebase from main frequently to stay close to the tip
- One logical concern per branch

---

## Rebasing

Rebase is the alternative to merging. Instead of creating a merge commit, rebase **replays** your commits on top of another branch.

### The concept

```
Before:
  main
    ↓
A ← B ← C ← F
          ↑
          └── D ← E  (feature-login)

After: git switch feature-login && git rebase main:
  main
    ↓
A ← B ← C ← F ← D' ← E'  (feature-login)
```

D and E are **rewritten** as D' and E' with new hashes (same changes, new parent). The branch now appears as if it was started from F, not C.

```bash
git switch feature-login
git rebase main
```

### What rebase does under the hood
1. Find the common ancestor of feature-login and main (C)
2. Save your commits (D, E) as patches
3. Reset feature-login to point at main's tip (F)
4. Re-apply your patches one by one as new commits (D', E')

### Pros and cons of rebase

| Pros | Cons |
|---|---|
| Clean linear history | Rewrites commit hashes — dangerous on shared branches |
| No noisy merge commits | Conflicts must be resolved commit-by-commit |
| Easy to read `git log` | More complex to understand and recover from |
| `git bisect` works better | Can confuse collaborators if rebased shared branch |

### The golden rule of rebasing
> **Never rebase commits that exist on a shared branch (main, develop).**

If a teammate has already based their work on your commits and you rebase them, they'll have a diverged history nightmare.

Safe to rebase: your local feature branch before opening a PR.

### Rebase vs Merge — When to use which

| Situation | Use |
|---|---|
| Feature branch → main, team uses squash | Squash merge |
| Feature branch → main, preserve history | Merge --no-ff |
| Update feature branch with main changes | Rebase (local, before pushing) |
| Merge two long-lived branches | Merge (preserves parallel context) |
| Integrating a hotfix | Merge or cherry-pick |

---

## Interactive Rebase

Interactive rebase lets you **rewrite, reorder, squash, edit, or drop** your own commits before sharing them.

```bash
git rebase -i HEAD~4    # Interactively rebase last 4 commits
git rebase -i main      # Interactively rebase everything after main's tip
```

This opens your editor with something like:
```
pick a3f5c1d feat: add login form
pick 8b2e4f1 fix typo
pick c9d3a7e wip: still broken
pick d4e5f6a working now, added tests

# Commands:
# pick   = use commit as-is
# reword = use commit, but edit message
# edit   = use commit, pause to amend
# squash = meld into previous commit (keeps both messages)
# fixup  = meld into previous commit (discards this message)
# drop   = remove commit entirely
```

### Common interactive rebase operations

**Squash messy WIP commits:**
```
pick a3f5c1d feat: add login form
fixup 8b2e4f1 fix typo
fixup c9d3a7e wip: still broken
fixup d4e5f6a working now, added tests
```
Result: one clean commit with the first message.

**Reorder commits:**
Just reorder the lines in the editor.

**Edit a commit's content:**
```
edit a3f5c1d feat: add login form
```
Git pauses at that commit. You edit files, `git add`, then:
```bash
git commit --amend
git rebase --continue
```

**Drop a commit entirely:**
```
drop c9d3a7e remove this commit
```

> **⚠️ Warning**: Interactive rebase rewrites history. Only do it on commits that haven't been pushed to shared branches.

---

## Detached HEAD State

Detached HEAD means HEAD points directly to a commit hash, not to a branch.

```bash
git checkout a3f5c1d     # Detaches HEAD
git checkout v1.0.0      # Checking out a tag also detaches
```

```
Normal state:        HEAD → main → commit F
Detached HEAD state: HEAD → commit C  (no branch)
```

### What you can do in detached HEAD
- Browse the code at that point in time
- Run the code, test things
- Make experimental commits

### The danger
If you make commits in detached HEAD and then switch away without creating a branch, those commits become orphaned — no branch points to them. They'll be garbage collected.

### Fix: create a branch from detached HEAD
```bash
git switch -c my-experiment    # Save your detached work
# or
git checkout -b my-experiment
```

### Getting back to normal
```bash
git switch main    # Re-attaches HEAD to main branch
```

---

## Feature Branch Workflow

The most common professional workflow. Every unit of work lives on its own branch.

```
main (protected, always deployable)
│
├── feature/user-auth
├── fix/session-timeout
├── chore/upgrade-react
└── feature/payment-integration
```

### Full lifecycle

```bash
# 1. Start from updated main
git switch main
git pull origin main

# 2. Create feature branch
git switch -c feature/user-auth

# 3. Work, commit often
git add src/auth.js
git commit -m "feat: add JWT validation middleware"

git add src/routes/login.js
git commit -m "feat: add login route"

# 4. Stay updated with main (optional, recommended for long branches)
git fetch origin
git rebase origin/main   # Or: git merge origin/main

# 5. Push branch to remote
git push -u origin feature/user-auth

# 6. Open Pull Request on GitHub

# 7. Address review comments
git add src/auth.js
git commit -m "fix: address PR review - validate token expiry"
git push    # PR auto-updates

# 8. After PR merges:
git switch main
git pull origin main       # Get the merged commit
git branch -d feature/user-auth    # Delete local branch
git push origin --delete feature/user-auth  # Delete remote branch
```

---

## Branch Lifecycle — Full Example

```
Scenario: fix a bug reported in issue #87

1. $ git switch main
2. $ git pull origin main
   Already up to date.

3. $ git switch -c fix/87-login-redirect
   Switched to a new branch 'fix/87-login-redirect'

4. (edit src/auth/callback.js)

5. $ git diff --staged  # (after git add)
   + return res.redirect('/dashboard');  # was redirecting to /

6. $ git add src/auth/callback.js
7. $ git commit -m "fix: correct login redirect to dashboard, closes #87"

8. $ git push -u origin fix/87-login-redirect

9. (open PR on GitHub, reviewer approves)

10. (PR merged via squash merge on GitHub)

11. $ git switch main
12. $ git pull origin main
13. $ git branch -d fix/87-login-redirect
```

---

## Beginner Mistakes

### ❌ Committing directly to main
Always work on a branch. `main` should be protected and only receive work via PRs.

### ❌ Creating branches off the wrong base
Always branch from an up-to-date `main`, not from someone else's feature branch (unless intentional).
```bash
git switch main && git pull && git switch -c feature/new   # Correct
```

### ❌ Not pulling before creating a branch
Your `main` may be behind the remote. New branch will be missing recent commits.

### ❌ Rebasing shared branches
Only rebase your local, unshared commits. Rebasing `main` = disaster.

### ❌ Leaving branches open after merging
Delete merged branches. Branch clutter makes repo navigation difficult.
```bash
git branch --merged main | grep -v main | xargs git branch -d
```

### ❌ Long-lived feature branches
The longer a branch lives, the more it diverges from main, and the harder it is to merge. Merge frequently or break large features into smaller pieces.

---

*Next: `05_Forking_and_Pull_Requests.md` — Open source contribution, PR workflows, code reviews*
