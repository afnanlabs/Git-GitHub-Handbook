# 05 — Forking and Pull Requests

> How open source contribution works, how PRs are used in teams, and how to be an effective reviewer and contributor.

---

## Table of Contents

1. [Why Forks Exist](#why-forks-exist)
2. [Fork vs Clone vs Branch](#fork-vs-clone-vs-branch)
3. [The Complete Fork Workflow](#the-complete-fork-workflow)
4. [Keeping Your Fork in Sync](#keeping-your-fork-in-sync)
5. [Contributing to Open Source — Step by Step](#contributing-to-open-source)
6. [Pull Request Anatomy](#pull-request-anatomy)
7. [Creating an Effective PR](#creating-an-effective-pr)
8. [Code Review — As a Reviewer](#code-review-as-a-reviewer)
9. [Code Review — As the Author](#code-review-as-the-author)
10. [Resolving Conflicts in a PR](#resolving-conflicts-in-a-pr)
11. [PR Merge Strategies](#pr-merge-strategies)
12. [PR Best Practices](#pr-best-practices)
13. [Common Mistakes](#common-mistakes)

---

## Why Forks Exist

The fundamental problem: you want to contribute to a project but you don't have write access to it.

Forks solve this by giving you **your own copy of the repo on GitHub**, where you have full control. You:
1. Make changes in your fork
2. Push commits to your fork
3. Open a Pull Request from your fork to the original repo

The original maintainers can review and merge without ever giving you direct access.

### Fork use cases
- Contributing to open source projects
- Using someone's project as a starting point for a derivative project
- Creating your own maintained version of an abandoned project

---

## Fork vs Clone vs Branch

| | Fork | Clone | Branch |
|---|---|---|---|
| What it creates | A new GitHub repo under your account | A local copy of a repo | A pointer within a repo |
| Lives on | GitHub servers | Your local machine | Local (or pushed to remote) |
| Write access | You own it | Depends on the original | Depends on repo permissions |
| Use case | Contributing without write access | Working locally on any repo | Feature isolation within a project you have access to |
| Relationship | Linked to original (upstream) | Linked to origin remote | Sibling of other branches |

---

## The Complete Fork Workflow

```
[Original Repo]                   [Your Fork on GitHub]
github.com/project/repo    →  Fork  →  github.com/you/repo
                                             ↓
                                          Clone
                                             ↓
                                    [Your Local Machine]
                                    git clone .../you/repo
                                             ↓
                                    Create branch, make changes
                                             ↓
                                    git push → your fork
                                             ↓
                                    Open PR: you/repo → project/repo
```

### Step-by-step setup

```bash
# Step 1: Fork on GitHub
# Click "Fork" button on github.com/project/repo

# Step 2: Clone YOUR fork (not the original)
git clone git@github.com:YOUR_USERNAME/repo.git
cd repo

# Step 3: Verify origin points to your fork
git remote -v
# origin  git@github.com:YOUR_USERNAME/repo.git (fetch)
# origin  git@github.com:YOUR_USERNAME/repo.git (push)

# Step 4: Add original repo as upstream
git remote add upstream git@github.com:ORIGINAL_PROJECT/repo.git

# Step 5: Verify
git remote -v
# origin    git@github.com:YOUR_USERNAME/repo.git (fetch)
# origin    git@github.com:YOUR_USERNAME/repo.git (push)
# upstream  git@github.com:ORIGINAL_PROJECT/repo.git (fetch)
# upstream  git@github.com:ORIGINAL_PROJECT/repo.git (push)
```

Now you have:
- `origin` = your fork (where you push your changes)
- `upstream` = the original project (where you pull updates from)

---

## Keeping Your Fork in Sync

The original project keeps moving. You need to regularly pull its changes into your fork.

```bash
# Fetch all changes from upstream
git fetch upstream

# Make sure you're on your main branch
git switch main

# Merge upstream's main into your local main
git merge upstream/main
# Or rebase (cleaner, if you prefer):
git rebase upstream/main

# Push updated main to your fork on GitHub
git push origin main
```

### When to sync
- Before creating any new feature branch
- When your PR shows conflicts with the base branch
- Periodically on long-running forks

### Automating fork sync on GitHub
GitHub has a built-in "Sync fork" button on your fork's Code tab (added 2022). It fetches upstream/main and fast-forwards your fork's main. Useful for quick syncs but doesn't handle your local machine.

---

## Contributing to Open Source

### Before writing code

1. **Read `CONTRIBUTING.md`** — almost all serious projects have one. It specifies code style, branch naming, PR requirements, issue linking.
2. **Check existing issues** — is someone already working on this? Comment on the issue before starting.
3. **Open an issue first** for non-trivial changes — get maintainer buy-in before investing effort.
4. **Read `CODE_OF_CONDUCT.md`** — understand community norms.

### Complete contribution workflow

```bash
# 1. Fork (on GitHub) + clone your fork
git clone git@github.com:YOU/project.git
cd project
git remote add upstream git@github.com:ORIGINAL/project.git

# 2. Sync with upstream main before starting
git fetch upstream
git switch main
git merge upstream/main

# 3. Create a focused branch
git switch -c fix/typo-in-readme
# Name it clearly: fix/, feature/, docs/, etc.

# 4. Make changes, commit with good messages
git add README.md
git commit -m "docs: fix typo in installation section"

# 5. Sync again if it took time (avoid conflicts)
git fetch upstream
git rebase upstream/main

# 6. Push to YOUR fork
git push origin fix/typo-in-readme

# 7. Open PR on GitHub
# From: YOU/project:fix/typo-in-readme
# To:   ORIGINAL/project:main
```

### Writing a PR for open source

Be specific and respectful. Maintainers are often volunteers. Structure:

```markdown
## What this PR does
Fixes a typo in the installation section of README.md.
"Instrallation" → "Installation" (line 23)

## Why
Noticed while setting up the project for the first time.

## Related issue
Closes #342
```

### If a maintainer requests changes
```bash
# Make the changes locally
git add README.md
git commit -m "docs: correct typo per review feedback"
git push origin fix/typo-in-readme
# PR updates automatically — no need to close and reopen
```

### If your PR gets rejected
Accept it gracefully. Maintainers know their project's roadmap. Don't take it personally.

---

## Pull Request Anatomy

Every PR on GitHub has:

```
Title:   fix: correct login redirect to dashboard
Labels:  bug, ready-for-review
Assignees: @you
Reviewers: @senior-dev, @tech-lead

Checks:  ✓ CI passed  ✓ Linting  ✓ Tests (47/47)

Diff:    +3 / -1 lines in src/auth/callback.js

Commits:
  a3f5c1d  fix: correct login redirect to dashboard
  8b2e4f1  fix: add test for redirect behavior

Files changed:
  src/auth/callback.js   +3 -1
  tests/auth.test.js     +12 -0

Conversation:
  @reviewer: "Should we also redirect unauthenticated users?"
  @you: "Good point — separate issue #92 for that."
  @reviewer: ✓ Approved

Base: main  ←  Compare: fix/login-redirect
```

---

## Creating an Effective PR

### PR title conventions
Follow the same Conventional Commits style:
```
feat: add dark mode toggle
fix: resolve null pointer on empty cart
chore: upgrade TypeScript to 5.4
docs: add API authentication guide
```

### PR description template (recommended)

```markdown
## Summary
[What does this PR do? Why?]

## Changes
- [List of changes]
- [Reference files modified]

## Related Issues
Closes #42

## Testing
- [x] Unit tests added/updated
- [x] Manual testing performed
- [ ] E2E tests updated (not applicable)

## Screenshots (if UI change)
[Before / After]

## Notes for reviewers
[Anything specific to pay attention to]
```

### PR size — the most important factor
| PR Size | Lines Changed | Reviewability |
|---|---|---|
| Ideal | < 200 | Easy to review thoroughly |
| Acceptable | 200–400 | Still manageable |
| Large | 400–800 | Reviewers will miss things |
| Too large | > 800 | Break it up |

Large PRs get rubber-stamped approvals or sit unreviewed for days. Small, focused PRs merge faster and get better feedback.

---

## Code Review — As a Reviewer

### What to look for
- **Correctness**: Does this do what it claims?
- **Edge cases**: What happens with null, empty, max values?
- **Security**: Any injection, auth bypass, data exposure risks?
- **Performance**: Any N+1 queries, unnecessary re-renders, missing indexes?
- **Readability**: Can you understand this code in 6 months?
- **Tests**: Are the tests meaningful, not just present?
- **Consistency**: Does it follow project conventions?

### Review comment types

**Nit (not blocking):**
```
nit: variable name could be more descriptive — `userToken` vs `t`
```

**Suggestion (inline, one-click apply):**
Use GitHub's "Suggest change" feature to propose exact replacements.

**Question (seeking understanding):**
```
Why is this check needed here? Wasn't it already validated upstream?
```

**Blocking concern:**
```
This will cause a SQL injection vulnerability. Input must be parameterized before hitting the DB.
```

### Review status options
- **Comment**: Feedback without blocking
- **Approve**: LGTM, ready to merge
- **Request Changes**: Must address issues before merging

### Be constructive
- Explain *why* something is an issue, not just that it is
- Offer alternatives when requesting changes
- Separate critical issues from nits
- Acknowledge what's done well

---

## Code Review — As the Author

### Responding to review comments

For every comment:
1. **Address it**: make the change, or
2. **Discuss it**: explain your reasoning if you disagree
3. **Mark as resolved**: click "Resolve conversation" once addressed

```bash
# After addressing review feedback:
git add src/auth.js
git commit -m "refactor: address PR review - extract validation"
git push   # PR auto-updates, reviewer is notified
```

### Re-requesting review
After pushing changes, click "Re-request review" next to the reviewer's name. Don't assume they'll notice the new push automatically.

### Never squash your fix commits yet
During review, keep commits separate — reviewers can see exactly what changed. Squash only after approval, before merging (if your team does that).

---

## Resolving Conflicts in a PR

When your PR shows "This branch has conflicts that must be resolved":

### Option 1: Resolve locally (preferred)
```bash
# Fetch latest main
git fetch origin

# Update your branch with rebase
git switch feature/your-branch
git rebase origin/main

# Resolve conflicts in each file
# (Git will pause at each conflicting commit)
git add conflicted-file.js
git rebase --continue

# Force-push (safe: it's your feature branch)
git push --force-with-lease origin feature/your-branch
```

### Option 2: Merge main into your branch
```bash
git switch feature/your-branch
git merge origin/main
# Resolve conflicts
git add .
git commit
git push origin feature/your-branch
```

Rebase produces cleaner history. Merge is simpler. Both are valid.

### Option 3: GitHub's web editor
GitHub has an in-browser conflict resolver for simple conflicts. Sufficient for single-file conflicts; avoid for complex multi-file conflicts.

---

## PR Merge Strategies

Available under the "Merge pull request" dropdown on GitHub:

### Merge commit
```
main: A ← B ← C ← M   (M has two parents: C and feature's last commit)
```
- Preserves full branch history
- Creates explicit merge commit
- `--no-ff` equivalent

### Squash and merge
```
main: A ← B ← C ← S   (S = all feature commits collapsed into one)
```
- Clean main history
- Lose individual feature commits
- Best when feature commits are messy WIPs

### Rebase and merge
```
main: A ← B ← C ← D' ← E'   (feature commits replayed on main)
```
- Linear history, no merge commit
- Individual commits preserved
- All commits get new hashes

### Which strategy to use
Most teams pick one and enforce it via repo settings. The most common professional choice:
- **Squash and merge**: if main history should be clean (one commit per feature)
- **Merge commit**: if branch history context matters for auditing
- Rebase: rarer, used by teams that are strict about linear history

---

## PR Best Practices

**Writing PRs:**
- One concern per PR — don't mix a bugfix with a refactor
- Write description before asking for review
- Self-review your diff before assigning reviewers (`git diff main..your-branch`)
- Link related issues
- Add screenshots for UI changes

**During review:**
- Respond within 24 hours on team projects
- Don't push unrelated changes while under review
- Don't merge your own PR (on team projects)

**After merge:**
- Delete the branch (GitHub has an option to auto-delete)
- Verify the deployment/CI ran correctly
- Close related issues if not auto-closed

---

## Common Mistakes

### ❌ Forking then cloning the original, not your fork
When you `git clone`, make sure the URL is `github.com/YOUR_USERNAME/repo`, not `github.com/ORIGINAL/repo`.

### ❌ Pushing to upstream instead of origin
With two remotes set up, always push to `origin` (your fork). Never push to `upstream` (you don't have write access and shouldn't push there directly).

### ❌ Opening a PR from main instead of a feature branch
If you commit to your fork's main and PR from that, you can't easily do follow-up work on a different PR simultaneously. Always branch.

### ❌ Letting a PR get stale
PRs that sit unreviewd diverge from main, accumulate conflicts, and lose context. Push for reviews actively.

### ❌ Not syncing upstream before a new PR
Start from a synced main every time. Stale base = guaranteed conflicts.

### ❌ Merging your own PRs without review
Even on solo projects, at minimum use a draft PR for CI validation. On team projects, always get a second set of eyes.

---

*Next: `06_Remotes_and_Origin.md` — How remote tracking works internally, SSH vs HTTPS, push/pull/fetch*
