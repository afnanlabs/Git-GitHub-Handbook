# 06 — Remotes and Origin

> How remote repositories work internally, the difference between fetch/pull/push, tracking branches, SSH vs HTTPS, and how collaboration flows.

---

## Table of Contents

1. [What a Remote Is](#what-a-remote-is)
2. [Origin and Upstream Conventions](#origin-and-upstream-conventions)
3. [Remote-Tracking Branches](#remote-tracking-branches)
4. [git fetch — Download Without Merging](#git-fetch)
5. [git pull — Fetch + Integrate](#git-pull)
6. [git push — Send Commits](#git-push)
7. [Tracking Branches](#tracking-branches)
8. [SSH vs HTTPS](#ssh-vs-https)
9. [Managing Multiple Remotes](#managing-multiple-remotes)
10. [How Collaboration Works Internally](#how-collaboration-works-internally)
11. [Syncing a Fork — Full Flow](#syncing-a-fork)
12. [Common Remote Issues](#common-remote-issues)

---

## What a Remote Is

A remote is simply a **named URL** pointing to another Git repository.

```bash
git remote add origin https://github.com/username/repo.git
```

This command creates an entry in `.git/config`:
```
[remote "origin"]
    url = https://github.com/username/repo.git
    fetch = +refs/heads/*:refs/remotes/origin/*
```

That's it. `origin` is just a shorthand for the URL. Git uses it every time you run `git push origin` or `git pull origin`.

### Key insight
Remotes are **local configuration** — they exist only in your `.git/config`. Two developers cloning the same repo will both have `origin` pointing to GitHub, but this is because they each set it up locally (clone does it automatically).

---

## Origin and Upstream Conventions

| Name | Convention | Set automatically? |
|---|---|---|
| `origin` | The repo you cloned from (usually your fork or your team repo) | Yes — by `git clone` |
| `upstream` | The original repo you forked from | No — you add it manually |

### When you clone directly (team project):
```
origin → github.com/company/project  (you have push access)
```

### When you fork (open source):
```
origin   → github.com/YOU/project    (your fork, you have push access)
upstream → github.com/PROJECT/project (original, read-only for you)
```

These are **conventions**, not enforced by Git. You could name them anything. But don't — every tool, tutorial, and teammate expects `origin` and `upstream`.

---

## Remote-Tracking Branches

When you fetch from a remote, Git creates local **remote-tracking branches** — read-only local references to the last known state of the remote's branches.

```bash
git fetch origin
```

Creates or updates:
```
refs/remotes/origin/main
refs/remotes/origin/develop
refs/remotes/origin/feature-login
```

These are displayed as `origin/main`, `origin/develop`, etc.

```bash
git branch -r    # Show remote-tracking branches
# origin/main
# origin/develop
# origin/feature-login

git log origin/main --oneline   # See what's on remote main (as of last fetch)
```

### What remote-tracking branches are NOT
- They are **not** your local branches
- You can't commit to `origin/main`
- They update **only when you fetch or pull**

This means `origin/main` can be stale — it shows the state of the remote as of your last fetch, not right now.

```bash
# Pattern: always fetch before comparing
git fetch origin
git log HEAD..origin/main --oneline   # What's on remote that I don't have?
git log origin/main..HEAD --oneline   # What I have that remote doesn't?
```

---

## git fetch

Downloads changes from a remote **without modifying your working directory or local branches**.

```bash
git fetch origin               # Fetch all branches from origin
git fetch origin main          # Fetch only the main branch
git fetch --all                # Fetch from all configured remotes
git fetch --prune              # Delete local remote-tracking refs for deleted remote branches
git fetch --tags               # Fetch tags
```

### What fetch actually does
1. Contacts the remote
2. Downloads new commit objects and data to `.git/objects/`
3. Updates remote-tracking branches (`origin/main`, etc.)
4. Does **nothing** to your local branches, staging area, or working directory

### Why use fetch alone?
- Preview what's changed before integrating
- Update your knowledge of remote state without risk
- Use with `git log` or `git diff` to analyze remote changes before merging

```bash
git fetch origin
git diff main origin/main    # See what's different
git log main..origin/main    # See commits you're missing
# Now decide how to merge/rebase
```

> **Habit**: `git fetch --all --prune` at the start of your day gives you a fresh view of all remote state.

---

## git pull

`git pull` = `git fetch` + `git merge` (or `git rebase` if configured)

```bash
git pull                    # Pull from tracked remote/branch
git pull origin main        # Explicit: fetch origin/main and merge into current
git pull --rebase           # Fetch + rebase instead of merge
git pull --ff-only          # Only update if fast-forward is possible (safe)
git pull --no-commit        # Merge but don't auto-commit
```

### The merge behavior of pull

If your local `main` and `origin/main` have diverged (both have commits the other doesn't), `git pull` creates a **merge commit**:

```
Before pull:
  local main:  A ← B ← C ← D  (you made D locally)
  origin/main: A ← B ← C ← E  (teammate pushed E)

After git pull:
  local main:  A ← B ← C ← D ← M
                             ↑
                             E
```

This creates messy "merge branch 'main' of github.com/..." commits in history.

### Prefer pull --rebase for cleaner history

```bash
git pull --rebase origin main
```

Result:
```
  local main:  A ← B ← C ← E ← D'
```

Your local commit D is replayed on top of E. Linear history, no merge commit.

Set as default:
```bash
git config --global pull.rebase true
```

### The `--ff-only` safety option

```bash
git pull --ff-only
```

This fails if the pull would require a merge or rebase. Use it when you want to ensure you only update if you're behind — not if there's any divergence. Great for scripts.

---

## git push

Sends your local commits to the remote.

```bash
git push origin main                  # Push main to origin
git push -u origin feature-login      # Push + set tracking (-u = --set-upstream)
git push                              # Push to configured tracking branch
git push origin --delete feature-x   # Delete remote branch
git push origin v2.0.0                # Push a tag
git push origin --tags                # Push all tags
git push --force-with-lease           # Safer force push (see below)
```

### The `-u` flag
`-u` (or `--set-upstream`) sets the **tracking relationship**. After this, bare `git push` and `git pull` know where to go without specifying remote and branch.

```bash
git push -u origin feature-login
# Now:
git push   # Same as git push origin feature-login
git pull   # Same as git pull origin feature-login
```

### Force pushing
```bash
git push --force          # ⚠️ Dangerous — overwrites remote, no safety check
git push --force-with-lease   # Safe force push — fails if remote has commits you haven't seen
```

**When force push is legitimate:**
- After rebasing your local feature branch (new commit hashes need to overwrite)
- Amending a commit that you've pushed to YOUR branch (before others based on it)

**When force push is NEVER acceptable:**
- On `main`, `develop`, or any shared branch
- When teammates have already pulled your old commits

```bash
# The professional force push
git rebase -i origin/main   # Clean up your feature branch
git push --force-with-lease origin feature-login   # Safe force push
```

---

## Tracking Branches

A tracking branch is a local branch configured to track a remote branch. When set, Git knows where to push and pull without you specifying each time.

```bash
# View tracking configuration
git branch -vv
# * feature-login  a3f5c1d [origin/feature-login] feat: add login form
#   main           8b2e4f1 [origin/main] chore: update deps

# Set tracking manually
git branch --set-upstream-to=origin/main main
git branch -u origin/feature-x feature-x   # Short form

# Create local branch tracking a remote branch
git checkout --track origin/feature-x
git switch --track origin/feature-x    # Modern
```

### What tracking enables
```bash
git status
# Your branch is behind 'origin/main' by 3 commits, use "git pull" to update it.
# Your branch is ahead of 'origin/main' by 2 commits, use "git push" to publish your local commits.
```

This status info only works when tracking is set up.

---

## SSH vs HTTPS

Two authentication methods for connecting to GitHub.

### HTTPS

```
https://github.com/username/repo.git
```

- Uses username + personal access token (PAT) for auth
- Requires entering credentials or using a credential manager
- Works everywhere (corporate firewalls, etc.)
- GitHub no longer accepts passwords — must use PAT or token

```bash
# Setup credential caching (macOS Keychain)
git config --global credential.helper osxkeychain

# Windows
git config --global credential.helper wincred

# Linux (in-memory for 1 hour)
git config --global credential.helper 'cache --timeout=3600'
```

### SSH

```
git@github.com:username/repo.git
```

- Uses SSH key pair for auth
- No password prompts after initial setup
- Keys must be added to GitHub account settings
- Some corporate firewalls block port 22 (SSH port)
- **Preferred for developers** — no credentials to manage after setup

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your@email.com"

# Start SSH agent and add key
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Copy public key to clipboard (macOS)
pbcopy < ~/.ssh/id_ed25519.pub

# Paste into GitHub → Settings → SSH and GPG keys → New SSH key

# Test connection
ssh -T git@github.com
# Hi username! You've successfully authenticated.
```

### Switching an existing repo from HTTPS to SSH

```bash
git remote -v
# origin  https://github.com/user/repo.git (fetch)

git remote set-url origin git@github.com:user/repo.git

git remote -v
# origin  git@github.com:user/repo.git (fetch)
```

### Comparison

| | SSH | HTTPS |
|---|---|---|
| Auth | SSH key pair | PAT/token |
| Credential prompts | Never (after setup) | Sometimes |
| Firewall issues | Possible (port 22) | Rare |
| Setup complexity | Medium | Low |
| Recommended for | Developers | CI/CD, quick access |

---

## Managing Multiple Remotes

Useful for: forked repos, mirroring, deploying to multiple environments.

```bash
git remote add origin git@github.com:you/repo.git
git remote add upstream git@github.com:original/repo.git
git remote add heroku git@heroku.com:myapp.git
git remote add staging git@github.com:company/repo-staging.git

git remote -v
# heroku   git@heroku.com:myapp.git (fetch)
# heroku   git@heroku.com:myapp.git (push)
# origin   git@github.com:you/repo.git (fetch)
# origin   git@github.com:you/repo.git (push)
# upstream git@github.com:original/repo.git (fetch)
# upstream git@github.com:original/repo.git (push)

# Push to multiple remotes (for mirroring):
git push origin main
git push heroku main   # Deploy
```

### Pushing to Heroku (example of non-GitHub remote)
```bash
heroku git:remote -a myapp-name   # Adds heroku remote automatically
git push heroku main               # Deploys to Heroku
```

---

## How Collaboration Works Internally

Understanding this prevents confusion about why pull/push behavior can surprise you.

### What happens when you push

1. Your client sends commit objects your remote doesn't have
2. Remote updates its ref (`refs/heads/main`) to point to your new commit
3. Remote rejects if your push isn't a fast-forward (you're behind)

```bash
# Push rejected example:
git push origin main
# To github.com:user/repo.git
#  ! [rejected]        main -> main (fetch first)
# error: failed to push some refs
# hint: Updates were rejected because the remote contains work that you do not have locally.
```

Fix: `git pull --rebase origin main`, then `git push`.

### What happens when you clone

1. Downloads all objects (commits, trees, blobs) from the remote
2. Creates the full `.git/` directory with entire history
3. Sets `origin` to the URL you cloned from
4. Creates local `main` tracking `origin/main`
5. Checks out `main` in your working directory

### What happens when teammates push

Nothing happens to your local repo automatically. Git is **not a live sync system**. You don't receive changes until you `git fetch` or `git pull`. This is why:
- You can be "behind" origin without knowing it
- You should `git fetch` at the start of your work session

---

## Syncing a Fork — Full Flow

```bash
# Initial setup (one time)
git remote add upstream git@github.com:ORIGINAL/repo.git

# Regular sync workflow
git fetch upstream                    # Download upstream changes
git switch main                       # Switch to your main
git merge upstream/main               # Merge upstream into local main
# or:
git rebase upstream/main              # Rebase local main onto upstream

git push origin main                  # Update your fork on GitHub

# Start new work from synced base
git switch -c feature/new-feature
```

### One-liner sync (merge approach)
```bash
git fetch upstream && git switch main && git merge upstream/main && git push origin main
```

---

## Common Remote Issues

### "Remote: Permission denied"
```bash
# Check you're using the right remote URL format
git remote -v

# If using HTTPS, your token may have expired. Generate new PAT on GitHub:
# Settings → Developer settings → Personal access tokens

# If using SSH, key may not be loaded:
ssh-add ~/.ssh/id_ed25519
ssh -T git@github.com   # Test connection
```

### "Push rejected: non-fast-forward"
```bash
# You're behind. Pull first.
git pull --rebase origin main
git push origin main
```

### "Diverged from remote"
```bash
# Your branch and the remote branch have different commits
git fetch origin
git log --oneline HEAD...origin/main   # See divergence

# Choose resolution:
git merge origin/main      # Merge (creates merge commit)
git rebase origin/main     # Rebase (linear history)
```

### "Remote branch deleted but still shows locally"
```bash
git fetch --prune   # Clean up stale remote-tracking branches
git remote prune origin  # Same effect
```

### Accidentally set wrong remote URL
```bash
git remote set-url origin git@github.com:CORRECT_USER/repo.git
```

---

*Next: `07_GitHub_Workflow_RealWorld.md` — How professional teams use GitHub day-to-day*
