# 08 — Debugging and Recovery

> The most important section for day-to-day development. Practical recovery procedures for everything that goes wrong.

---

## Table of Contents

1. [The Golden Rule Before Doing Anything](#golden-rule)
2. [git reflog — Your Primary Safety Net](#git-reflog)
3. [Undoing an Uncommitted Change](#undoing-uncommitted-change)
4. [Undoing a Staged Change](#undoing-staged-change)
5. [Undoing the Last Commit (Not Pushed)](#undoing-last-commit-local)
6. [Undoing Multiple Commits (Not Pushed)](#undoing-multiple-commits-local)
7. [Undoing a Pushed Commit (Safe Method)](#undoing-pushed-commit-safe)
8. [Undoing a Pushed Commit (Force Push Method)](#undoing-pushed-commit-force)
9. [Recovering a Deleted Branch](#recovering-deleted-branch)
10. [Recovering Stashed Work](#recovering-stashed-work)
11. [Fixing a Merge Gone Wrong](#fixing-merge-gone-wrong)
12. [Fixing a Rebase Gone Wrong](#fixing-rebase-gone-wrong)
13. [Recovering Lost Files](#recovering-lost-files)
14. [Fixing Wrong Author/Email on Commits](#fixing-wrong-author)
15. [Fixing a Corrupt Repository](#fixing-corrupt-repository)
16. [The Reset Decision Tree](#reset-decision-tree)
17. [Revert vs Reset — When to Use Which](#revert-vs-reset)
18. [Common Disaster Scenarios](#disaster-scenarios)

---

## Golden Rule

> **Before doing anything drastic: run `git reflog` and `git status`.**

Almost nothing is truly lost in Git within ~90 days. The reflog records every movement of HEAD, so you can almost always get back to where you were.

**Before any recovery attempt:**
```bash
git status         # What is the current state?
git reflog         # Where has HEAD been?
git log --oneline  # What does current history look like?
```

When in doubt: **don't run `git reset --hard` or `git clean -fd` until you understand what they'll destroy.**

---

## git reflog — Your Primary Safety Net

Reflog records every time HEAD or a branch pointer moved. Think of it as Git's internal undo history.

```bash
git reflog
```

Output:
```
a3f5c1d HEAD@{0}: commit: feat: add payment flow
8b2e4f1 HEAD@{1}: checkout: moving from main to feature/payment
c9d3a7e HEAD@{2}: merge feature/auth: Fast-forward
d4e5f6a HEAD@{3}: reset: moving to HEAD~1
e5f6a7b HEAD@{4}: commit: wip: broken state
```

Each entry shows:
- `a3f5c1d`: the commit hash HEAD was at
- `HEAD@{0}`: positional reference (0 = current)
- What action moved HEAD there

### Using reflog to recover

```bash
# Recover to any point in reflog
git checkout HEAD@{3}                    # Detached HEAD at that state
git checkout -b recovered HEAD@{3}      # Create branch at that state
git reset --hard HEAD@{3}               # Move current branch to that state

# Branch-specific reflog
git reflog show feature/auth
```

> **Note**: Reflog is local and not pushed to GitHub. It doesn't exist in fresh clones. Reflog entries expire after 90 days (configurable).

---

## Undoing an Uncommitted Change

### Discard changes to a specific file (working directory)
```bash
git restore src/app.js          # Discard changes, restore from last commit
git checkout -- src/app.js      # Old syntax, same effect
```

### Discard ALL uncommitted changes
```bash
git restore .                   # Discard all working directory changes
git checkout -- .               # Old syntax
```

> **⚠️ Warning**: This is permanent. There is no undo for discarding uncommitted changes (they were never in a commit). Check `git diff` first.

### Discard untracked files (files Git has never seen)
```bash
git clean -n       # Dry run: show what would be deleted
git clean -f       # Delete untracked files
git clean -fd      # Delete untracked files and directories
```

---

## Undoing a Staged Change

You ran `git add` but want to unstage without losing the changes:

```bash
git restore --staged src/app.js    # Unstage specific file
git restore --staged .             # Unstage everything
git reset HEAD src/app.js          # Old syntax, same effect
git reset                          # Unstage everything (old syntax)
```

The file reverts to "modified, unstaged" — your changes are still there.

---

## Undoing the Last Commit (Not Pushed)

### Keep changes staged (undo commit, keep work ready to recommit)
```bash
git reset --soft HEAD~1
```
Use when: wrong commit message, want to add more to the same commit.

### Keep changes in working directory (undo commit, unstage changes)
```bash
git reset HEAD~1           # --mixed is default
git reset --mixed HEAD~1   # Explicit
```
Use when: committed too early, want to rethink what to stage.

### Discard the commit AND all changes
```bash
git reset --hard HEAD~1
```
Use when: the commit was a mistake and you want the code gone too.

> **⚠️ Warning**: `--hard` permanently discards your working directory changes. Run `git diff HEAD~1` first to see what you'd be losing.

### Just fix the commit message or add a forgotten file
```bash
# Fix message only:
git commit --amend -m "corrected commit message"

# Add a forgotten file:
git add forgotten-file.js
git commit --amend --no-edit   # Keep same message

# Both message and content:
git add forgotten-file.js
git commit --amend
# (editor opens with pre-filled message, modify if needed)
```

> **⚠️ Warning**: Amend changes the commit hash. If you've pushed this commit, you'll need to force-push. Only amend before pushing, or on your own branches.

---

## Undoing Multiple Commits (Not Pushed)

```bash
# Undo last 3 commits, keep changes staged
git reset --soft HEAD~3

# Undo last 3 commits, keep changes in working dir
git reset --mixed HEAD~3

# Undo last 3 commits, discard all changes
git reset --hard HEAD~3

# Reset to a specific commit hash
git reset --hard a3f5c1d

# Interactive rebase to reorder/edit/drop commits
git rebase -i HEAD~3
```

---

## Undoing a Pushed Commit (Safe Method)

When commits are already on a shared branch, use `git revert`. It creates a NEW commit that undoes the changes — preserving history.

```bash
# Revert the last commit
git revert HEAD

# Revert a specific commit
git revert a3f5c1d

# Revert a range of commits (oldest first, newest listed first in command)
git revert HEAD~2..HEAD     # Revert last 2 commits

# Revert without auto-committing (stage the reversal, commit manually)
git revert --no-commit HEAD
git revert --no-commit HEAD~1
git commit -m "revert: undo payment changes due to bug in production"
```

Why revert is safe:
- Doesn't rewrite history
- Teammates' local repos stay compatible
- The revert is auditable (you can see what was undone and why)

---

## Undoing a Pushed Commit (Force Push Method)

Only use on **your own feature branches** that no one else has pulled.

```bash
# Check that nobody else has this branch checked out first

# Reset locally to before the bad commit
git reset --hard HEAD~1

# Force push (--force-with-lease is safer than --force)
git push --force-with-lease origin feature/my-branch
```

`--force-with-lease` fails if anyone has pushed to the remote branch since your last fetch. It's a safeguard against overwriting others' work.

> **⚠️ Never force-push `main`, `develop`, or any branch that others are actively using.**

---

## Recovering a Deleted Branch

### If deleted locally (remote still exists)
```bash
git fetch origin
git switch -c feature/deleted-branch origin/feature/deleted-branch
# or:
git checkout --track origin/feature/deleted-branch
```

### If deleted both locally and remotely — via reflog
```bash
git reflog
# Find the commit hash where your branch was last pointing
# Example: a3f5c1d HEAD@{3}: checkout: moving from feature/deleted-branch to main

git switch -c feature/restored a3f5c1d
```

### Finding lost commit hashes with fsck
```bash
git fsck --lost-found
# Lists all "dangling commits" (not reachable from any branch or tag)
# These are commits from deleted branches or reset operations
```

Dangling commits are written to `.git/lost-found/commit/`.

---

## Recovering Stashed Work

```bash
git stash list
# stash@{0}: WIP on main: add payment
# stash@{1}: WIP on feature/auth: half-done

git stash pop                      # Apply latest stash, remove from stash list
git stash apply stash@{1}          # Apply specific stash, keep in stash list
git stash show -p stash@{0}        # Inspect contents before applying
```

### Recovering a dropped stash
If you accidentally ran `git stash drop` or `git stash clear`:

```bash
git fsck --no-reflog | grep commit | awk '{print $3}' | xargs git show --stat
# Shows all unreachable commits — your stash objects are among them
```

Stashes are stored as special commits. `git fsck` can find them even after dropping.

```bash
# Once you find the stash commit hash:
git stash apply <hash>
```

---

## Fixing a Merge Gone Wrong

### Abort before completing the merge
```bash
git merge --abort
# Returns to pre-merge state cleanly
```

### Undo a completed merge (not pushed)
```bash
git reset --hard HEAD~1     # If it created one merge commit
# or find the pre-merge commit in reflog:
git reflog
git reset --hard HEAD@{n}   # Where n is the entry just before the merge
```

### Undo a completed merge (pushed to shared branch)
```bash
# Find the merge commit hash
git log --merges --oneline

# Revert the merge commit (specify parent 1 = the branch you merged into)
git revert -m 1 <merge-commit-hash>
git push origin main
```

The `-m 1` flag tells revert which side of the merge is the "mainline" (parent 1 = the branch you were on when you merged).

---

## Fixing a Rebase Gone Wrong

### Abort mid-rebase
```bash
git rebase --abort     # Return to pre-rebase state
```

### Undo a completed rebase (not pushed)
Rebase moves your branch pointer and creates new commits. The old commits still exist in reflog.

```bash
git reflog
# Find the commit hash your branch pointed to BEFORE the rebase
# It will be labeled: "rebase finished" or "checkout" before the rebase started

git reset --hard HEAD@{n}    # n = entry before the rebase began
```

Or by branch reflog:
```bash
git reflog show feature/my-branch
# Find the entry before the rebase
git reset --hard feature/my-branch@{2}
```

---

## Recovering Lost Files

### File deleted and committed
```bash
# Find which commit deleted it
git log --diff-filter=D --name-only -- path/to/file.js

# Restore it from the commit before deletion
git checkout <commit-before-deletion> -- path/to/file.js
git add path/to/file.js
git commit -m "restore: recover accidentally deleted file"
```

### File lost in a `git reset --hard`
```bash
git fsck --lost-found
# Check .git/lost-found/other/ for blobs (file contents)
```

### File deleted by `git clean -f`
If it was tracked by Git (committed at some point), find it in history:
```bash
git log --all --full-history -- path/to/file.js
git show <commit-hash>:path/to/file.js > recovered-file.js
```

If it was untracked (never committed): it's gone. Git never knew about it.

---

## Fixing Wrong Author/Email on Commits

### Fix the last commit only
```bash
git commit --amend --author="Correct Name <correct@email.com>"
```

### Fix multiple recent commits
```bash
git rebase -i HEAD~5    # Interactive rebase last 5 commits
# Mark commits as "edit"

# For each marked commit, when Git pauses:
git commit --amend --author="Correct Name <correct@email.com>" --no-edit
git rebase --continue
```

### Fix globally going forward
```bash
git config --global user.name "Correct Name"
git config --global user.email "correct@email.com"
```

> **Note**: Fixing old commits that are already pushed to shared branches requires force-pushing, which will affect collaborators. Avoid this situation by setting your git config correctly from the start.

---

## Fixing a Corrupt Repository

### Symptoms
```
error: object file .git/objects/... is empty
fatal: loose object is corrupt
```

### Recovery steps
```bash
# Step 1: Check integrity
git fsck --full

# Step 2: Try to repair from packfiles
git gc --aggressive

# Step 3: If you have a remote (GitHub), the safest fix is:
cd ..
git clone git@github.com:user/repo.git repo-recovered
# Copy any uncommitted changes from the corrupt working directory
# Continue from the fresh clone
```

Corruption is rare in practice. Usually caused by disk failure, out-of-disk during write, or forceful shutdown during Git operations.

---

## Reset Decision Tree

```
Did you already push to a shared branch?
    YES → Use git revert (creates new commit, safe)
    NO  → Use git reset

Are you undoing staged changes only (no commit yet)?
    YES → git restore --staged (or git reset HEAD)

Which reset mode?
    Keep changes staged →    git reset --soft HEAD~N
    Keep changes unstaged → git reset --mixed HEAD~N  (default)
    Discard changes →       git reset --hard HEAD~N   (⚠️ destructive)
```

---

## Revert vs Reset — When to Use Which

| Scenario | Command | Why |
|---|---|---|
| Undo last commit, keep changes | `git reset --soft HEAD~1` | Local only, efficient |
| Undo pushed commit on shared branch | `git revert HEAD` | Safe, preserves history |
| Remove a specific old commit from shared branch | `git revert <hash>` | Safe |
| Clean up messy local commits before PR | `git rebase -i HEAD~N` | Professional habit |
| Undo pushed commit on your own branch | `git reset --hard HEAD~1` + force push | Your branch, your rules |
| Rollback production to known good state | `git revert <merge-commit> -m 1` | Safe, auditable |

### The mental model

**`reset`** moves a branch pointer backward. History is rewritten. Previous commits still exist in reflog but are no longer on the branch.

**`revert`** moves a branch pointer forward with a new "undo" commit. Previous commits remain. History tells the full story including the mistake and its correction.

Use `revert` by default for anything shared. Use `reset` for local-only cleanup.

---

## Common Disaster Scenarios

### "I committed to main instead of my feature branch"
```bash
# Create the branch that should have had this commit
git branch feature/correct-branch

# Reset main to before your commit
git reset --hard HEAD~1

# Now go to the correct branch (it still has the commit)
git switch feature/correct-branch
```

### "I merged the wrong branch"
```bash
# If not pushed:
git reset --hard HEAD~1   # Remove merge commit

# If pushed:
git revert -m 1 <merge-commit-hash>
git push origin main
```

### "I accidentally ran git reset --hard"
```bash
git reflog
# Find HEAD@{1} or earlier where you were before the reset
git reset --hard HEAD@{1}
```

### "I deleted a branch with important work"
```bash
git reflog | grep "feature/my-branch"
# Find the last commit hash on that branch
git checkout -b feature/my-branch <hash>
```

### "git push --force overwrote my teammate's work"
```bash
# Your teammate needs to:
git reflog show origin/main   # Find the overwritten commit
# Recover is tricky — they need to have that commit locally

# Prevention: always use --force-with-lease, never --force
```

### "I committed secrets (API key, password)"
```bash
# This requires history rewriting:
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch path/to/secret-file' \
  --prune-empty --tag-name-filter cat -- --all

git push origin --force --all   # Push rewritten history

# But also:
# 1. Revoke the secret IMMEDIATELY (rotate the API key, change password)
# 2. Assume it was already scraped — act as if compromised
# 3. For complex cases, use: https://github.com/newren/git-filter-repo
```

> **Critical**: Removing secrets from Git history does NOT make them safe if the repo was ever public or accessible to others. Always rotate/revoke the credential immediately.

---

*Next: `09_Advanced_Git.md` — Rebase in depth, cherry-pick, bisect, hooks, submodules, Git internals*
