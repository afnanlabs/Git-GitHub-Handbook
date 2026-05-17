# 09 — Advanced Git

> Rebase in depth, cherry-pick, bisect, hooks, submodules, and Git internals.  
> Topics you need once you're past basic workflow and need precise control.

---

## Table of Contents

1. [Rebasing — Deep Explanation](#rebasing-deep)
2. [Interactive Rebase — Full Reference](#interactive-rebase-full)
3. [Cherry-Pick](#cherry-pick)
4. [git bisect — Binary Search for Bugs](#git-bisect)
5. [Git Hooks](#git-hooks)
6. [Submodules](#submodules)
7. [Git Internals — Object Storage](#git-internals)
8. [Blobs, Trees, and Commits](#blobs-trees-commits)
9. [Packfiles](#packfiles)
10. [Worktrees](#worktrees)
11. [git filter-repo](#git-filter-repo)
12. [Signing Commits (GPG)](#signing-commits)

---

## Rebasing — Deep Explanation

### What rebase actually does, step by step

Given this state:
```
main:    A ← B ← C ← F
                  ↑
feature:          └── D ← E
```

`git switch feature && git rebase main` does:

1. Find common ancestor: C
2. Save D and E as patch files (temporary)
3. Move feature pointer to F (same as main's tip)
4. Apply patch D on top of F → creates D' (new hash, same content)
5. Apply patch E on top of D' → creates E' (new hash, same content)

Result:
```
main:    A ← B ← C ← F
                       ↑
feature:               └── D' ← E'
```

D' and E' are **new commits** with the same changes but new hashes. The original D and E still exist in the object store but are no longer reachable from any branch (they'll be garbage collected).

### When rebase creates conflicts

If D modifies a line that F also modified, Git will pause at D' with a conflict:

```bash
git rebase main
# CONFLICT (content): Merge conflict in src/auth.js

# Resolve conflict in the file, then:
git add src/auth.js
git rebase --continue

# To skip this commit (rare):
git rebase --skip

# To abort completely:
git rebase --abort
```

Unlike a merge conflict (which happens once), rebase conflicts can happen at each commit being replayed. For 5 commits, you might resolve 5 separate conflicts.

### Rebasing onto a different point

```bash
# Rebase only part of a branch
git rebase --onto main feature/base feature/child
# Takes commits unique to feature/child (not in feature/base)
# and replays them on top of main
```

This is useful for splitting a branch that was accidentally based on another feature branch instead of main.

### Upstream rebase (pulling with rebase)

```bash
git pull --rebase origin main
```

This is: fetch origin/main, then rebase your local commits on top of it. Result is linear, clean history.

Set as default:
```bash
git config --global pull.rebase true
```

---

## Interactive Rebase — Full Reference

Interactive rebase gives you a UI (in your configured editor) to rewrite a series of commits.

```bash
git rebase -i HEAD~5        # Edit last 5 commits
git rebase -i main          # Edit all commits since branching from main
git rebase -i a3f5c1d       # Edit all commits after this hash
```

### The editor interface

```
pick a3f5c1d feat: add login form
pick 8b2e4f1 fix typo in login form
pick c9d3a7e wip: broken
pick d4e5f6a finally working
pick e5f6a7b add tests

# Rebase f2a3b4c..e5f6a7b onto f2a3b4c (5 commands)
#
# Commands:
# p, pick   = use commit
# r, reword = use commit, but edit the commit message
# e, edit   = use commit, but stop for amending
# s, squash = meld into previous commit (keep all messages)
# f, fixup  = meld into previous commit (discard this message)
# x, exec   = run command (the rest of the line) using shell
# b, break  = stop here (continue rebase later with 'git rebase --continue')
# d, drop   = remove commit
# l, label  = label current HEAD with a name
```

### Common interactive rebase operations

**Squash WIP commits into one clean commit:**
```
pick a3f5c1d feat: add login form
fixup 8b2e4f1 fix typo in login form
fixup c9d3a7e wip: broken
fixup d4e5f6a finally working
pick e5f6a7b add tests
```
Result: 2 commits. The WIPs are absorbed.

**Reword a commit message:**
```
reword a3f5c1d feat: add login form
```
Git will pause and open your editor to edit the message.

**Edit a commit's content:**
```
edit c9d3a7e fix: correct auth logic
```
Git pauses at that commit. You can:
```bash
git add forgotten-thing.js
git commit --amend --no-edit
git rebase --continue
```

**Reorder commits:** Just move the lines.

**Drop a commit:**
```
drop d4e5f6a remove this mistake entirely
```

**Split one commit into two:**
```
edit a3f5c1d feat: add login form and payment  # too much in one commit
```
Git pauses. Then:
```bash
git reset HEAD~1           # Unstage the commit's changes
git add src/login.js
git commit -m "feat: add login form"
git add src/payment.js
git commit -m "feat: add payment form"
git rebase --continue      # Tell rebase we're done with this step
```

---

## Cherry-Pick

Copy a specific commit from one branch to another, without merging the whole branch.

```bash
git cherry-pick a3f5c1d            # Apply one commit to current branch
git cherry-pick a3f5c1d 8b2e4f1    # Apply multiple commits
git cherry-pick a3f5c1d..8b2e4f1   # Apply a range (exclusive of first)
git cherry-pick a3f5c1d^..8b2e4f1  # Apply a range (inclusive of first)
git cherry-pick --no-commit a3f5c1d  # Apply changes without committing
```

### Real-world use cases

**Backporting a fix to an older release:**
```bash
# Bug was fixed in main
# Need to apply fix to v2.x maintenance branch

git switch release/v2.x
git cherry-pick a3f5c1d   # Hash of the fix commit from main
git push origin release/v2.x
```

**Pulling one commit from a colleague's branch:**
```bash
git fetch origin
git cherry-pick origin/feature/payment:a3f5c1d
```

**Rescuing commits from a wrong branch:**
```bash
# You committed to main instead of your feature branch
# Create feature branch
git switch -c feature/correct

# Go back to main and undo
git switch main
git reset --hard HEAD~1
```

(Or cherry-pick is the other direction: pick the commit onto the correct branch, then drop it from main)

### Cherry-pick conflicts
If cherry-pick conflicts:
```bash
# Resolve conflict
git add conflicted-file.js
git cherry-pick --continue

# Abort
git cherry-pick --abort
```

### What cherry-pick creates
A new commit with the same changes but different hash (new parent, new timestamp). The original commit is not moved or deleted.

---

## git bisect — Binary Search for Bugs

Find exactly which commit introduced a bug using binary search. If you have 1000 commits to search through, bisect finds the culprit in ~10 steps.

### Workflow

```bash
# Start bisect
git bisect start

# Mark current (latest) as bad
git bisect bad

# Mark a known-good commit (could be a tag, hash, or relative ref)
git bisect good v2.0.0

# Git checks out a commit in the middle
# Test your code. If bug exists:
git bisect bad
# If bug doesn't exist:
git bisect good

# Repeat until Git says:
# "a3f5c1d is the first bad commit"

# End bisect (returns to original branch)
git bisect reset
```

### Automated bisect

If you have a test script that exits 0 (pass) or non-zero (fail):

```bash
git bisect start
git bisect bad HEAD
git bisect good v2.0.0
git bisect run npm test   # Or any script
# Git runs this at each step and decides good/bad automatically
```

### Real-world scenario

```bash
git bisect start
git bisect bad HEAD        # App crashes now
git bisect good v1.5.0     # v1.5.0 was fine

# Git checks out commit between HEAD and v1.5.0
# You test the app

git bisect good            # No crash here
# Git bisects further

git bisect bad             # Crash here
# ... continues ...

# Git output:
# d4e5f6a is the first bad commit
# Author: Dev Name <dev@example.com>
# Date: Mon May 5 11:23:01 2025
#
#     refactor: change authentication middleware order

git bisect reset
# Now you know exactly which refactor broke it
```

---

## Git Hooks

Hooks are scripts Git runs automatically at specific points in the workflow. Located in `.git/hooks/`.

### Client-side hooks

| Hook | When it runs | Use cases |
|---|---|---|
| `pre-commit` | Before commit is created | Lint, format, run tests |
| `prepare-commit-msg` | Before commit message editor | Auto-fill message template |
| `commit-msg` | After message is entered | Validate message format |
| `post-commit` | After commit is created | Notifications |
| `pre-push` | Before push to remote | Run full test suite |
| `pre-rebase` | Before rebase | Safety check |
| `post-merge` | After merge completes | Run `npm install` if package.json changed |
| `post-checkout` | After checkout | Set up environment |

### Creating a hook

```bash
# Create pre-commit hook
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/sh
npm run lint
exit $?   # Non-zero exit = commit blocked
EOF
chmod +x .git/hooks/pre-commit
```

### pre-commit: lint before every commit
```bash
#!/bin/sh
# Run ESLint on staged JS files
STAGED_FILES=$(git diff --cached --name-only --diff-filter=ACM | grep ".jsx\?$")

if [ -n "$STAGED_FILES" ]; then
    echo "Running ESLint..."
    npx eslint $STAGED_FILES
    if [ $? -ne 0 ]; then
        echo "Lint failed. Fix errors before committing."
        exit 1
    fi
fi
```

### commit-msg: enforce conventional commits
```bash
#!/bin/sh
COMMIT_MSG_FILE=$1
COMMIT_MSG=$(cat $COMMIT_MSG_FILE)

PATTERN="^(feat|fix|docs|style|refactor|test|chore|ci|perf|revert)(\(.+\))?: .{1,72}"

if ! echo "$COMMIT_MSG" | grep -qE "$PATTERN"; then
    echo "Invalid commit message format."
    echo "Use: type(scope): description"
    echo "Example: feat(auth): add login rate limiting"
    exit 1
fi
```

### Shareable hooks with Husky

`.git/hooks/` isn't committed to the repo (it's inside `.git/`). Use [Husky](https://typicode.github.io/husky/) to share hooks with your team:

```bash
npm install husky --save-dev
npx husky init

# Creates .husky/ directory (committed to repo)
# Add hooks:
echo "npm run lint" > .husky/pre-commit
echo "npm test" > .husky/pre-push
```

### lint-staged — Only lint staged files

```bash
npm install lint-staged --save-dev
```

```json
// package.json
{
  "lint-staged": {
    "*.js": ["eslint --fix", "prettier --write"],
    "*.ts": ["tsc --noEmit", "eslint --fix"]
  }
}
```

```bash
# .husky/pre-commit
npx lint-staged
```

---

## Submodules

Submodules embed one Git repository inside another as a specific commit reference.

### When to use submodules
- Shared libraries across multiple projects
- External dependencies you want to vendor
- Large monorepos with independent components

### Adding a submodule

```bash
git submodule add https://github.com/user/shared-lib.git libs/shared-lib
# Creates .gitmodules file
# Creates a special "submodule" entry in .git
```

### Cloning a repo with submodules

```bash
# Option 1: Clone then initialize submodules
git clone https://github.com/user/project.git
cd project
git submodule init
git submodule update

# Option 2: Clone with submodules in one step
git clone --recurse-submodules https://github.com/user/project.git
```

### Updating submodules

```bash
# Update all submodules to their latest commit
git submodule update --remote

# Update a specific submodule
git submodule update --remote libs/shared-lib

# Pull changes in submodules too
git pull --recurse-submodules
```

### Submodule caveats
- Submodules track a **specific commit**, not a branch
- Forgetting to push a submodule update before the parent project is a common trap
- Collaborators often forget to run `git submodule update` after pulling
- **Alternative**: package managers (npm, pip, cargo) are often better than submodules for dependencies

---

## Git Internals — Object Storage

Everything Git stores is an **object** — a compressed file in `.git/objects/`.

Objects are identified by their SHA-1 hash (40 hex chars). The first 2 chars are the directory name, the remaining 38 are the filename:

```
.git/objects/
└── a3/
    └── f5c1d2e8b9f4a1c7d3e6b2f9a4c1d8e3f7b2a1
```

### Four object types

1. **blob** — raw file content
2. **tree** — directory listing (filename → blob/tree references)
3. **commit** — tree + parent + metadata
4. **tag** — named reference with message

### Inspecting objects

```bash
git cat-file -t a3f5c1d   # Type of object
# commit

git cat-file -p a3f5c1d   # Pretty-print contents
# tree 7d3e1b9...
# parent 8f2a1b3...
# author Jane Dev <jane@example.com> 1682345678 +0530
# committer Jane Dev <jane@example.com> 1682345678 +0530
#
# feat: add login form

git cat-file -p 7d3e1b9   # Look at the tree
# 100644 blob 2f9a4c1... README.md
# 100644 blob 8b3d7e2... src/app.js
# 040000 tree 4a1c7d3... tests
```

---

## Blobs, Trees, and Commits

### How a commit stores the entire project

```
commit a3f5c1d
│
└── tree 7d3e1b9  (root directory)
    ├── blob 2f9a4c1 → "# My Project\n..."  (README.md contents)
    ├── blob 8b3d7e2 → "const app = ..."    (src/app.js contents)
    └── tree 4a1c7d3  (tests/ directory)
        └── blob 9f4b2e1 → "describe('app'..." (tests/app.test.js contents)
```

### Why hashing enables deduplication

If `README.md` hasn't changed between commit A and commit B, they both reference the same blob object `2f9a4c1`. The content is stored exactly once. This is how Git is space-efficient despite storing full snapshots.

### The hash → immutability connection

If you change one character in a file, the blob hash changes. Since the tree references the blob by hash, the tree hash changes. Since the commit references the tree hash, the commit hash changes. This cascade is why:
- You can't secretly modify Git history (the hash would no longer match)
- Amending a commit changes its hash and all descendants
- Rebase creates new commits (new hashes), not modifications

---

## Packfiles

Git initially stores objects as "loose objects" (individual files). Over time, or on push/fetch, Git packs them into **packfiles** for efficiency.

```bash
git gc             # Run garbage collection + packing
git gc --aggressive # More thorough (slower) packing

# View pack stats
git count-objects -v
# count: 12            ← loose objects
# packs: 1             ← packfiles
# in-pack: 2847        ← objects in packfiles
```

Packfiles use **delta compression** — storing the difference between similar objects rather than full copies. A series of text file versions is compressed extremely well.

You rarely need to think about packfiles directly — Git manages them automatically.

---

## Worktrees

Check out multiple branches simultaneously in separate directories without cloning.

```bash
# Add a worktree for hotfix work
git worktree add ../hotfix-branch hotfix/critical-bug

# Now you have:
# /projects/myapp        (your normal working directory, on feature/auth)
# /projects/hotfix-branch (separate directory, on hotfix/critical-bug)

# List worktrees
git worktree list

# Remove a worktree when done
git worktree remove ../hotfix-branch
```

### When worktrees are useful
- Working on a hotfix while in the middle of a feature branch (no stashing needed)
- Running two different versions of the app simultaneously for comparison
- Long-running background processes that need a specific branch checked out

---

## git filter-repo

The modern tool for rewriting Git history across an entire repository. Replaces the deprecated `git filter-branch`.

```bash
# Install: pip install git-filter-repo

# Remove a file from ALL history
git filter-repo --path src/secrets.txt --invert-paths

# Rename a path in history
git filter-repo --path-rename old/path:new/path

# Change all commit author emails
git filter-repo --email-callback '
  return email if email != b"old@email.com" else b"new@email.com"
'

# Extract a subdirectory as its own repo (with history)
git filter-repo --subdirectory-filter src/module-name
```

> **Warning**: After filter-repo, ALL commit hashes change. Inform all collaborators, who will need to re-clone. This should be done on a private copy first.

---

## Signing Commits (GPG)

Sign commits cryptographically to prove they came from you. GitHub shows a "Verified" badge on signed commits.

```bash
# Generate a GPG key (if you don't have one)
gpg --full-generate-key

# List your keys
gpg --list-secret-keys --keyid-format=long

# Get your key ID (the long hex after rsa4096/)
# Example: rsa4096/3AA5C34371567BD2

# Configure Git to use your key
git config --global user.signingkey 3AA5C34371567BD2

# Sign commits automatically
git config --global commit.gpgsign true

# Or sign manually per commit
git commit -S -m "signed commit message"

# Export public key to add to GitHub
gpg --armor --export 3AA5C34371567BD2
# Copy to GitHub → Settings → SSH and GPG keys → New GPG key
```

### Why sign commits?
Anyone can set `git config user.name "Linus Torvalds"` and make commits that appear to be from Linus. GPG signing cryptographically verifies that a commit was actually made by the key holder. Required in some enterprise and government environments.

### SSH signing (simpler alternative, newer)
```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
```

---

*Next: `10_Deployment_and_Team_Workflows.md` — CI/CD, deployment platforms, release workflows*
