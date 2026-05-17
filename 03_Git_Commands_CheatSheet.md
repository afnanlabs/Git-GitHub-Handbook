# 03 — Git Commands Cheat Sheet

> Organized, practical, searchable. Every command with syntax, explanation, real-world use, and warnings.

---

## Quick Navigation

- [Setup & Config](#setup--config)
- [Creating & Cloning Repos](#creating--cloning-repos)
- [Staging & Committing](#staging--committing)
- [Viewing Status & History](#viewing-status--history)
- [Branching](#branching)
- [Merging & Rebasing](#merging--rebasing)
- [Remote Management](#remote-management)
- [Undoing Mistakes](#undoing-mistakes)
- [Stashing](#stashing)
- [Tags](#tags)
- [Searching & Comparing](#searching--comparing)
- [Cleaning](#cleaning)
- [Advanced One-Liners](#advanced-one-liners)

---

## Setup & Config

### `git config`
Configure Git identity and behavior.

```bash
# Global (applies to all repos on your machine)
git config --global user.name "Jane Dev"
git config --global user.email "jane@example.com"
git config --global core.editor "code --wait"    # VS Code as editor
git config --global init.defaultBranch main       # Set default branch name

# View all config
git config --list
git config --global --list

# View specific value
git config user.name

# Set per-repo (override global)
git config user.email "work@company.com"    # run inside the repo, no --global
```

**Real-world use**: Different email for personal vs work projects. Set per-repo config to use your work email in company projects.

> **⚠️ Warning**: If your email doesn't match a verified GitHub email, commits won't appear in the contribution graph.

---

## Creating & Cloning Repos

### `git init`
Initialize a new Git repository in the current directory.

```bash
git init                    # Creates .git/ in current directory
git init my-project         # Creates my-project/ with .git/ inside
git init --bare repo.git    # Bare repo (for servers, no working directory)
```

**Real-world use**: Starting a new project locally before creating the remote.

### `git clone`
Download a repository from a remote URL.

```bash
git clone https://github.com/user/repo.git           # Clone via HTTPS
git clone git@github.com:user/repo.git               # Clone via SSH
git clone https://github.com/user/repo.git my-name   # Clone into custom folder name
git clone --depth=1 https://github.com/user/repo.git # Shallow clone (latest snapshot only)
git clone --branch develop https://github.com/u/r    # Clone and checkout specific branch
```

**Real-world use**: Starting work on an existing project. Always prefer SSH for repos you'll push to (no password prompts).

> **Note**: `--depth=1` is useful for CI environments or when you only need the latest state and don't care about history.

---

## Staging & Committing

### `git add`
Move changes from working directory to staging area.

```bash
git add filename.js            # Stage specific file
git add src/                   # Stage all files in a directory
git add .                      # Stage all changes in current directory and below
git add *.js                   # Stage all JS files
git add -A                     # Stage all changes (new, modified, deleted) everywhere
git add -p                     # Interactive patch mode — choose hunks to stage
git add -u                     # Stage modifications and deletions only (not new files)
```

**Real-world use**: `git add -p` is the professional's tool — stage exactly the lines you want, nothing more. Essential for keeping commits focused.

> **⚠️ Warning**: `git add .` stages everything including files you may not want committed. Always check `.gitignore` first and run `git status` after.

### `git commit`
Create a snapshot from the staging area.

```bash
git commit -m "feat: add user login"              # Commit with inline message
git commit                                        # Opens editor for message
git commit -am "fix: typo in header"             # Stage tracked files + commit (skips git add)
git commit --amend                               # Modify the last commit (message or content)
git commit --amend -m "corrected message"        # Amend with new message
git commit --amend --no-edit                     # Amend without changing message
git commit --allow-empty -m "trigger CI"         # Commit with no file changes
```

**Commit message conventions (Conventional Commits):**
```
type(scope): description

feat: add payment integration
fix: resolve null pointer in auth
docs: update API reference
style: format button CSS
refactor: extract validation logic
test: add unit tests for UserService
chore: upgrade dependencies
ci: add GitHub Actions workflow
```

**Real-world use**: `--amend` is used constantly to fix the last commit before pushing. Never amend pushed commits on shared branches.

> **⚠️ Warning**: `git commit -am` only stages *tracked* files. New (untracked) files are silently ignored. Use `git add` for new files.

### `git rm`
Remove files from tracking and optionally from disk.

```bash
git rm filename.js              # Delete file and stage the deletion
git rm --cached filename.js     # Untrack file but keep it on disk (useful for .gitignore fixes)
git rm -r --cached node_modules/ # Untrack directory (add to .gitignore first)
```

**Real-world use**: `git rm --cached` is used when you accidentally committed a file that should be in `.gitignore`. Remove from tracking, add to `.gitignore`, commit.

### `git mv`
Rename or move a tracked file.

```bash
git mv old-name.js new-name.js
git mv src/utils.js lib/helpers.js
```

**Real-world use**: Git can track renames when done via `git mv` — it preserves history better than manually renaming and doing `git add`.

---

## Viewing Status & History

### `git status`
Show current state of working directory and staging area.

```bash
git status           # Full status output
git status -s        # Short/compact output
git status -sb       # Short + branch info
```

Output interpretation:
```
On branch main
Changes to be committed:        ← Staged (will be in next commit)
  modified: src/app.js

Changes not staged for commit:  ← Modified but not staged
  modified: src/auth.js

Untracked files:                ← New files Git doesn't know about
  src/utils.js
```

**Habit**: Run `git status` constantly. Before adding, before committing, before pushing.

### `git log`
View commit history.

```bash
git log                                     # Full log
git log --oneline                           # One line per commit
git log --oneline --graph --all             # Visual branch graph
git log --oneline -10                       # Last 10 commits
git log --author="Jane"                     # Filter by author
git log --since="2 weeks ago"              # Filter by date
git log --until="2025-01-01"              # Filter before date
git log --grep="fix"                       # Filter by message keyword
git log -p                                 # Show patch (diff) for each commit
git log --stat                             # Show file change stats
git log -- src/auth.js                     # History of a specific file
git log main..feature-branch               # Commits in feature not in main
git log --follow -- old-name.js            # Follow renames in history
```

**Real-world use**: `git log --oneline --graph --all` gives you a visual overview of the entire branch structure. Alias it:
```bash
git config --global alias.lg "log --oneline --graph --all --decorate"
git lg  # Now works as shortcut
```

### `git diff`
Show what changed between zones or commits.

```bash
git diff                        # Working dir vs staging (unstaged changes)
git diff --staged               # Staging vs last commit (what will be committed)
git diff HEAD                   # Working dir vs last commit (all changes)
git diff main..feature-branch   # Compare two branches
git diff a3f5c1d..8b2e4f1       # Compare two commits
git diff -- src/app.js          # Diff a specific file only
git diff --stat                 # Summary of changes (no line-by-line)
git diff --word-diff            # Highlight word-level changes
```

**Habit**: Always run `git diff --staged` before committing to verify exactly what you're committing.

### `git show`
Show details of a specific commit.

```bash
git show                    # Show last commit
git show a3f5c1d            # Show specific commit
git show HEAD~2             # Show commit 2 before HEAD
git show v1.0.0             # Show tagged commit
git show a3f5c1d:src/app.js # Show file content at that commit
```

---

## Branching

### `git branch`
List, create, and manage branches.

```bash
git branch                  # List local branches
git branch -a               # List all branches (local + remote)
git branch -r               # List remote branches only
git branch feature-login    # Create new branch (don't switch)
git branch -d feature-login # Delete merged branch (safe)
git branch -D feature-login # Force delete (unmerged)
git branch -m old new       # Rename branch
git branch -m main master   # Rename current branch (no old name needed)
git branch -v               # Show last commit on each branch
git branch --merged         # Branches already merged into current
git branch --no-merged      # Branches not yet merged
```

### `git checkout` / `git switch`
Switch branches or restore files.

```bash
# Modern way (Git 2.23+)
git switch main                    # Switch to existing branch
git switch -c feature-login        # Create and switch to new branch
git switch -                       # Switch to previous branch

# Old way (still works, still common)
git checkout main
git checkout -b feature-login
git checkout -

# Special: check out a specific commit (detached HEAD)
git checkout a3f5c1d

# Restore a file from another branch
git checkout main -- src/config.js  # Get config.js from main into current branch
```

> **Note**: `git switch` and `git restore` were introduced to split `git checkout`'s responsibilities. Both the old and new syntax work identically. Learn both — you'll see both in the wild.

### `git restore`
Restore working directory or staged files.

```bash
git restore filename.js          # Discard uncommitted changes to a file
git restore .                    # Discard ALL uncommitted changes
git restore --staged filename.js # Unstage a file (keep changes in working dir)
git restore --source=HEAD~2 file # Restore file to state 2 commits ago
```

> **⚠️ Warning**: `git restore filename.js` permanently discards your uncommitted edits. There is no undo. Make sure you don't need those changes.

---

## Merging & Rebasing

### `git merge`
Merge changes from one branch into another.

```bash
git merge feature-login          # Merge feature-login into current branch
git merge --no-ff feature-login  # Force merge commit (no fast-forward)
git merge --squash feature-login # Squash all commits into staged changes (then commit manually)
git merge --abort                # Abort a merge in progress
git merge --continue             # Continue after resolving conflicts
```

### `git rebase`
Replay commits on top of another branch.

```bash
git rebase main                   # Rebase current branch onto main
git rebase -i HEAD~3              # Interactive rebase: edit last 3 commits
git rebase -i main                # Interactive rebase onto main
git rebase --abort                # Abort a rebase in progress
git rebase --continue             # Continue after resolving conflicts
git rebase --skip                 # Skip current conflicting commit
```

See `04_Branching_and_Merging.md` for detailed merge/rebase guidance.

---

## Remote Management

### `git remote`
Manage remote repository connections.

```bash
git remote -v                              # List remotes and their URLs
git remote add origin git@github.com:u/r  # Add remote named origin
git remote add upstream git@github.com:o/r # Add upstream (for forks)
git remote rename origin new-name         # Rename a remote
git remote remove origin                  # Remove a remote connection
git remote set-url origin new-url         # Change URL of existing remote
git remote get-url origin                 # Show URL of a remote
```

### `git fetch`
Download remote changes without merging.

```bash
git fetch origin               # Fetch all changes from origin
git fetch origin main          # Fetch only the main branch
git fetch --all                # Fetch from all remotes
git fetch --prune              # Remove local refs to deleted remote branches
```

**Real-world use**: Use `git fetch` before `git pull` to preview what's coming. `git fetch` is always safe — it never changes your working directory.

### `git pull`
Fetch + merge remote changes.

```bash
git pull                         # Pull from tracking remote/branch
git pull origin main             # Pull explicitly
git pull --rebase                # Pull + rebase instead of merge (cleaner history)
git pull --ff-only               # Only pull if fast-forward is possible
```

> **Note**: `git pull --rebase` is preferred by many teams to keep history linear. Set globally: `git config --global pull.rebase true`

### `git push`
Send commits to remote.

```bash
git push origin main                 # Push main to origin
git push -u origin feature-login     # Push and set tracking (-u = --set-upstream)
git push                             # Push to tracked remote branch
git push origin --delete feature-x  # Delete remote branch
git push origin v1.0.0               # Push a tag
git push origin --tags               # Push all tags
git push --force-with-lease          # Safer force push (fails if remote changed)
```

> **⚠️ Warning**: NEVER `git push --force` on shared branches (main, develop). It rewrites remote history and will break teammates' repos. Use `--force-with-lease` if you must force push, and only on your own branches.

---

## Undoing Mistakes

### `git reset`
Move HEAD (and optionally staging/working dir) to a different commit.

```bash
# Modes:
git reset --soft HEAD~1   # Undo last commit, keep changes staged
git reset HEAD~1          # Undo last commit, keep changes in working dir (default: --mixed)
git reset --hard HEAD~1   # Undo last commit, DISCARD all changes

# By hash:
git reset --soft a3f5c1d
git reset --hard a3f5c1d

# Unstage a specific file (safe, doesn't touch working dir):
git reset HEAD src/app.js
```

| Mode | Commit | Staging | Working Dir |
|---|---|---|---|
| `--soft` | Moved | Unchanged | Unchanged |
| `--mixed` (default) | Moved | Cleared | Unchanged |
| `--hard` | Moved | Cleared | Cleared |

> **⚠️ Warning**: `--hard` discards working directory changes permanently. Only use on commits that haven't been pushed to shared branches.

### `git revert`
Create a new commit that undoes a previous commit (safe for shared history).

```bash
git revert HEAD             # Undo last commit (creates new "revert" commit)
git revert a3f5c1d          # Undo a specific commit
git revert HEAD~3..HEAD     # Revert last 3 commits
git revert --no-commit HEAD # Stage the reverting changes without committing
```

**When to use revert vs reset:**
- `revert`: already pushed to shared branch — safe, preserves history
- `reset`: local only, or on your own branch before pushing — faster but destructive

### `git reflog`
View the history of where HEAD has pointed — your safety net.

```bash
git reflog              # Show HEAD movement history
git reflog show main    # Show branch-specific reflog
```

Output:
```
a3f5c1d HEAD@{0}: commit: feat: add login
8b2e4f1 HEAD@{1}: reset: moving to HEAD~1
c9d3a7e HEAD@{2}: commit: add payment
```

```bash
# Restore a "lost" commit or branch
git checkout HEAD@{2}          # Check out that state
git branch recovered-work HEAD@{2}  # Create branch to save it
git reset --hard HEAD@{1}      # Jump back to that state
```

Reflog entries expire after ~90 days. This is your emergency recovery tool.

---

## Stashing

### `git stash`
Temporarily shelve uncommitted changes.

```bash
git stash                          # Stash current changes (tracked files)
git stash -u                       # Stash including untracked files
git stash -m "wip: login form"     # Stash with a label
git stash list                     # View all stashes
git stash pop                      # Apply latest stash + remove it
git stash apply                    # Apply latest stash + keep it
git stash apply stash@{2}          # Apply a specific stash
git stash drop stash@{0}           # Delete a specific stash
git stash clear                    # Delete all stashes
git stash show -p stash@{0}        # Show what's in a stash
git stash branch new-branch        # Create branch from stash
```

**Real-world use**: Colleague needs urgent review, but you're mid-feature. `git stash`, switch branches, help them, switch back, `git stash pop`.

> **⚠️ Warning**: Stash is not a backup. Stashes can be dropped accidentally. If work matters, commit to a WIP branch instead.

---

## Tags

### `git tag`
Create reference points in history (versions, releases).

```bash
git tag                             # List tags
git tag v1.0.0                      # Lightweight tag
git tag -a v1.0.0 -m "Release 1.0" # Annotated tag (preferred)
git tag -a v1.0.0 a3f5c1d           # Tag a specific commit
git show v1.0.0                     # Show tag details
git tag -d v1.0.0                   # Delete local tag
git push origin v1.0.0              # Push single tag
git push origin --tags              # Push all tags
git push origin --delete v1.0.0     # Delete remote tag
git checkout v1.0.0                 # Check out tagged state (detached HEAD)
```

---

## Searching & Comparing

### `git grep`
Search for a string across all tracked files.

```bash
git grep "TODO"                 # Find all TODOs
git grep -n "function login"    # With line numbers
git grep -i "error"             # Case-insensitive
git grep "TODO" HEAD~5          # Search in a past commit
```

### `git log -S` (Pickaxe)
Find commits that added or removed a specific string.

```bash
git log -S "password_hash"          # Which commit introduced/removed this?
git log -G "regex_pattern"          # Same but with regex
```

**Real-world use**: "When was this function introduced?" or "Who removed the error handler?"

### `git blame`
See who last modified each line of a file.

```bash
git blame src/auth.js
git blame -L 20,40 src/auth.js    # Only lines 20-40
git blame --follow src/auth.js    # Follow through renames
```

---

## Cleaning

### `git clean`
Remove untracked files from working directory.

```bash
git clean -n              # Dry run — show what would be deleted
git clean -f              # Delete untracked files
git clean -fd             # Delete untracked files and directories
git clean -fX             # Delete only ignored files (from .gitignore)
git clean -fdx            # Delete everything untracked (including ignored)
```

> **⚠️ Warning**: `git clean -fd` is destructive and cannot be undone. Always run with `-n` first to preview.

---

## Advanced One-Liners

```bash
# Undo last commit but keep changes staged
git reset --soft HEAD~1

# Stage all and commit with one line
git add -A && git commit -m "message"

# Amend without opening editor
git commit --amend --no-edit

# See what's different between your branch and main
git log main..HEAD --oneline

# Copy a single commit from another branch
git cherry-pick a3f5c1d

# Find when a bug was introduced
git bisect start
git bisect bad HEAD
git bisect good v1.0.0

# Show all aliases
git config --global --list | grep alias

# Delete all local branches that have been merged to main
git branch --merged main | grep -v "main" | xargs git branch -d

# Show compact log with dates
git log --pretty=format:"%h %ad | %s%d [%an]" --date=short

# Show files changed in last commit
git diff-tree --no-commit-id -r --name-only HEAD

# Pull all remotes and prune stale branches
git fetch --all --prune

# Create a patch file from a commit
git format-patch HEAD~1

# Apply a patch file
git apply mypatch.patch
```

---

## Useful Aliases to Set Up

```bash
git config --global alias.s "status -sb"
git config --global alias.lg "log --oneline --graph --all --decorate"
git config --global alias.undo "reset --soft HEAD~1"
git config --global alias.unstage "restore --staged"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.aliases "config --global --list | grep alias"
```

---

*Next: `04_Branching_and_Merging.md` — Branches, merges, rebases, conflicts, visual diagrams*
