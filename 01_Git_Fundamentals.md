# 01 — Git Fundamentals

> Core theory, mental models, and architecture.  
> Read this to understand WHY Git behaves the way it does — not just HOW to use it.

---

## Table of Contents

1. [What Git Is](#what-git-is)
2. [Why Git Exists](#why-git-exists)
3. [Version Control Systems — History](#version-control-systems)
4. [Snapshots vs Diffs](#snapshots-vs-diffs)
5. [Git Architecture — The Three Zones](#git-architecture)
6. [The .git Directory](#the-git-directory)
7. [Working Directory](#working-directory)
8. [Staging Area (Index)](#staging-area)
9. [Repository and Commit History](#repository-and-commit-history)
10. [Commits and Commit Hashes](#commits-and-commit-hashes)
11. [HEAD](#head)
12. [Local vs Remote Repositories](#local-vs-remote-repositories)
13. [Git Object Model](#git-object-model)
14. [Beginner Mental Model Mistakes](#beginner-mental-model-mistakes)

---

## What Git Is

Git is a **distributed version control system** (DVCS). It tracks changes to files over time and allows multiple people to collaborate on the same codebase without overwriting each other's work.

Key properties:
- **Distributed**: Every developer has the entire repository history locally. No single server is required to work.
- **Fast**: Almost all operations are local (no network needed).
- **Reliable**: Uses SHA-1/SHA-256 hashing to verify data integrity.
- **Branching-first**: Branches are central to Git's design, not an afterthought.

Git was created by **Linus Torvalds in 2005** for managing Linux kernel development after a licensing dispute with the existing tool (BitKeeper). The design goals: speed, simplicity, strong support for non-linear development (thousands of parallel branches), and full distribution.

---

## Why Git Exists

Before Git, teams used centralized version control (CVS, SVN). Problems:

| Problem | How Git Solves It |
|---|---|
| Server goes down → no one can commit | Every local clone is a full repo |
| Branching is slow and risky | Branches are just pointers, near-free |
| Merge conflicts are catastrophic | Built-in conflict detection and resolution |
| History can be corrupted | SHA hashing detects any data corruption |
| Everyone steps on each other's work | Isolated branches + pull request reviews |

---

## Version Control Systems

### Generation 1 — Local VCS
Track file changes locally. No collaboration. RCS (Revision Control System) is the classic example. Stores patch sets on disk.

### Generation 2 — Centralized VCS (CVCS)
Single server stores all versions. Clients check out files from that server.
- Examples: **CVS**, **SVN (Subversion)**, **Perforce**
- Problem: Server is a single point of failure. Network required for most operations. Branching is expensive.

### Generation 3 — Distributed VCS (DVCS)
Every client mirrors the full repository, including history.
- Examples: **Git**, **Mercurial**
- Result: offline work, fast local operations, resilient to server failure, cheap branching.

---

## Snapshots vs Diffs

This is the most important mental model in Git.

### How SVN (diff-based) thinks:
```
Version 1: [file A]
Version 2: [file A + change 1]
Version 3: [file A + change 1 + change 2]
```
SVN stores the *delta* (what changed) between versions. To reconstruct version 3, it replays all deltas from version 1.

### How Git (snapshot-based) thinks:
```
Commit 1: snapshot of all files at time 1
Commit 2: snapshot of all files at time 2
Commit 3: snapshot of all files at time 3
```
Each Git commit is a **complete snapshot** of every tracked file at that moment.

**But doesn't that waste space?**  
No. Git is smart: if a file hasn't changed between commits, it stores a reference (pointer) to the previous snapshot of that file — not a duplicate. Changed files get new blobs. Unchanged files share blob objects.

**Why this matters:**
- Switching branches is instant — Git just loads a different snapshot
- Time travel (checkout old commit) is trivial
- History is self-contained — any commit can reconstruct the full file tree
- Performance doesn't degrade over time like delta-replay systems

---

## Git Architecture

### The Three Zones (Most Important Mental Model)

```
┌─────────────────────┐     git add      ┌──────────────────┐     git commit    ┌──────────────────┐
│                     │ ───────────────► │                  │ ────────────────► │                  │
│  Working Directory  │                  │  Staging Area    │                  │   Repository     │
│  (your actual files)│ ◄─────────────── │  (Index)         │ ◄──────────────── │   (.git/objects) │
│                     │  git restore     │                  │  git reset HEAD   │                  │
└─────────────────────┘                  └──────────────────┘                  └──────────────────┘
```

| Zone | What it is | Where it lives |
|---|---|---|
| **Working Directory** | Your actual files on disk as you edit them | Your project folder |
| **Staging Area (Index)** | A prepared snapshot of what will be committed | `.git/index` |
| **Repository** | All committed snapshots, the full history | `.git/objects/` |

### Flow of a typical change:
1. You edit `app.js` in your working directory
2. `git add app.js` → stages it (marks it for the next snapshot)
3. `git commit -m "fix login bug"` → creates a permanent snapshot in the repository

### Why the staging area exists
The staging area lets you craft precise commits. You might edit 5 files but only want 2 of them in the next commit. Stage only those 2, commit, then stage the remaining 3.

This is how professionals write clean commit history: one logical change per commit.

---

## The .git Directory

When you run `git init`, Git creates a `.git/` folder in your project root. This folder **is** the repository. Your project files outside it are just the working directory.

```
.git/
├── HEAD              ← points to current branch or commit
├── config            ← repo-level git config
├── index             ← staging area (binary file)
├── objects/          ← all commits, trees, blobs stored here
│   ├── pack/         ← packed objects for efficiency
│   └── info/
├── refs/
│   ├── heads/        ← local branches (each file = one branch)
│   │   ├── main
│   │   └── feature-login
│   ├── tags/         ← tags
│   └── remotes/      ← remote tracking branches
│       └── origin/
└── logs/
    ├── HEAD          ← reflog for HEAD
    └── refs/heads/   ← reflog for each branch
```

> **⚠️ Warning**: Never manually edit files inside `.git/` unless you know exactly what you're doing. You can corrupt your repository.

> **Note**: Deleting `.git/` completely removes Git tracking from a project. The files remain but all history is gone. This is irreversible.

---

## Working Directory

The working directory is your project folder — the files you see and edit.

Git tracks files in one of four states:

| State | Meaning |
|---|---|
| **Untracked** | Git has never seen this file. Not in any snapshot. |
| **Tracked, unmodified** | In last commit, not changed since. |
| **Tracked, modified** | In last commit, but you've changed it. |
| **Staged** | Modified (or new) and marked for the next commit. |

Check the state of everything with:
```bash
git status
```

---

## Staging Area

The staging area (also called the **index**) is a preparatory zone between your edits and your commits.

### Why professionals love it
```bash
# Edit 3 files: auth.js, styles.css, README.md
# But this commit should only be about auth

git add auth.js
git commit -m "fix: correct JWT expiration logic"

# Now stage the rest separately
git add styles.css
git commit -m "style: update button colors"

git add README.md
git commit -m "docs: update setup instructions"
```

This produces a clean, reviewable history instead of one messy "updated stuff" commit.

### Partial staging
You can even stage specific lines within a file:
```bash
git add -p filename.js   # Interactive patch mode — choose which hunks to stage
```
This is used by experienced developers to separate unrelated edits within the same file into different commits.

---

## Repository and Commit History

The repository is the full graph of all commits, stored in `.git/objects/`.

### Commit graph structure
```
A ← B ← C ← D  (main)
          ↑
          └── E ← F  (feature-branch)
```

Each commit points to its parent. This forms a Directed Acyclic Graph (DAG).

- Commits are immutable — once made, they don't change.
- The "history" is just traversing parent pointers backwards from the current commit.
- Branches are just named pointers into this graph.

### Viewing history
```bash
git log                          # Full log
git log --oneline                # Compact
git log --oneline --graph --all  # Visual graph of all branches
git log --author="Jane"          # Filter by author
git log --since="2 weeks ago"    # Filter by date
git log -p                       # Show diff with each commit
```

---

## Commits and Commit Hashes

### What a commit is
A commit object contains:
- **Tree**: a snapshot of the project's file structure at that moment
- **Parent(s)**: reference(s) to the previous commit(s)
- **Author**: name, email, timestamp
- **Committer**: name, email, timestamp (can differ from author after rebase)
- **Message**: the commit message you wrote

### Commit hash (SHA)
Every commit is identified by a **SHA-1 hash** — a 40-character hexadecimal string generated from the commit's content:

```
commit a3f5c1d2e8b9f4a1c7d3e6b2f9a4c1d8e3f7b2a1
Author: Jane Dev <jane@example.com>
Date:   Mon May 12 14:32:01 2025

    feat: add user authentication
```

Because the hash is derived from the content + parent hash + metadata:
- Any change to the commit (even the message) produces a completely different hash
- Hashes are globally unique — no two commits in any Git repo should ever collide
- You can reference commits by the first 7 characters (usually enough for uniqueness): `a3f5c1d`

### Why immutability matters
When you "amend" a commit or "rebase," Git isn't modifying the commit. It's creating a **new commit** with a new hash. The old commit still exists in `.git/objects/` until garbage collection runs.

This is why `git reflog` can recover "lost" commits — they aren't actually deleted immediately.

---

## HEAD

HEAD is a special pointer that tells Git **where you currently are** in the repository.

### Normal state — HEAD points to a branch
```
HEAD → main → commit D
```
When you commit, `main` moves to the new commit, and HEAD follows.

```bash
cat .git/HEAD
# ref: refs/heads/main
```

### Detached HEAD — HEAD points directly to a commit
```
HEAD → commit C  (no branch)
```
This happens when you:
```bash
git checkout a3f5c1d    # checkout a specific commit hash
git checkout v1.0.0     # checkout a tag
```

In detached HEAD state:
- You can look around, run code, experiment
- If you make commits, they have no branch label — they'll be lost after switching away
- Fix it by creating a branch: `git checkout -b my-experiment`

```bash
# Check if you're in detached HEAD
git status
# HEAD detached at a3f5c1d
```

---

## Local vs Remote Repositories

| Concept | Description |
|---|---|
| **Local repo** | Your full repository on your own machine, in `.git/` |
| **Remote repo** | A copy of the repository on another machine (e.g., GitHub) |
| **origin** | The default nickname Git gives to the remote you cloned from |
| **upstream** | Conventionally, the original repo you forked from |

### The key insight
**There is no magical "real" repository.** GitHub doesn't own your code — it just hosts another copy. All copies are equal in terms of Git structure. GitHub's copy is typically the "source of truth" by team convention, not by technical requirement.

### How local and remote stay in sync

```bash
git fetch origin    # Download changes from remote, don't apply them
git pull origin main # Fetch + merge remote main into current branch
git push origin main # Send your commits to remote
```

Remote-tracking branches (e.g., `origin/main`) are **local read-only references** that show you the last known state of the remote.

---

## Git Object Model

Git stores everything as four types of objects in `.git/objects/`:

| Object Type | What it stores |
|---|---|
| **blob** | Contents of a single file |
| **tree** | A directory listing (filenames + references to blobs/subtrees) |
| **commit** | Metadata + pointer to a tree + parent commit(s) |
| **tag** | Pointer to a commit with extra metadata |

### Example: what a commit really is

```
commit a3f5c1d
├── tree 7d3e1b9           ← root directory snapshot
│   ├── blob 2f9a4c1 → src/app.js
│   ├── blob 8b3d7e2 → src/auth.js
│   └── tree 4a1c7d3 → tests/
│       └── blob 9f4b2e1 → tests/auth.test.js
├── parent 8f2a1b3         ← previous commit
├── author Jane Dev
└── message "feat: add auth"
```

This structure means:
- Two commits that share unchanged files will point to the **same blob objects** — zero duplication
- The entire project state is reconstructable from a single commit hash

---

## Beginner Mental Model Mistakes

### ❌ Mistake 1: "git add ." means I saved my work"
`git add .` only moves files to the staging area. Nothing is saved to history until `git commit`. You can lose staged changes if you reset.

### ❌ Mistake 2: "Branches are copies of my code"
Branches are just pointers. No files are duplicated. Creating a branch costs almost nothing.

### ❌ Mistake 3: "Deleting a branch deletes my commits"
Deleting a branch just removes the label. Commits still exist in object storage until garbage collected. You can recover them via `git reflog`.

### ❌ Mistake 4: "git pull is always safe"
`git pull` = `git fetch` + `git merge`. The merge part can create merge commits or conflicts. Use `git fetch` first to see what's coming, then decide how to integrate.

### ❌ Mistake 5: "My commit hash is the same as my colleague's for the same change"
No. Commit hashes include timestamp and author, so even identical changes produce different hashes on different machines.

### ❌ Mistake 6: "git reset deleted my commits forever"
Commits persist in reflog for ~90 days. Use `git reflog` to find and restore them. See `08_Debugging_and_Recovery.md`.

### ❌ Mistake 7: "The remote is the 'real' repository"
GitHub is just another copy. Your local repo is equally valid and complete. The remote is "authoritative" only by team convention.

---

*Next: `02_GitHub_Fundamentals.md` — GitHub's interface, features, and collaboration model*
