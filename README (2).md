# Git & GitHub — Complete Practical Reference System

> A deeply structured knowledge base for developers who want long-term mastery,  
> real-world workflow confidence, and debugging fluency — not a beginner tutorial.

---

## How to Use This Reference

This is **not** a course to read linearly. It is a reference system to consult repeatedly.

| Situation | Go to |
|---|---|
| You forget how Git stores data | `01_Git_Fundamentals.md` |
| You need a quick command | `03_Git_Commands_CheatSheet.md` |
| You're confused about branching | `04_Branching_and_Merging.md` |
| You're contributing to open source | `05_Forking_and_Pull_Requests.md` |
| You're confused about push/pull/fetch | `06_Remotes_and_Origin.md` |
| You're working in a team | `07_GitHub_Workflow_RealWorld.md` |
| Something broke and you need to fix it | `08_Debugging_and_Recovery.md` |
| You want to rebase, cherry-pick, bisect | `09_Advanced_Git.md` |
| You're deploying or setting up CI/CD | `10_Deployment_and_Team_Workflows.md` |
| You don't know what a term means | `11_Git_Glossary.md` |

---

## File Index

```
Git-GitHub/
│
├── README.md                          ← You are here
├── 01_Git_Fundamentals.md             ← What Git is, how it thinks, mental models
├── 02_GitHub_Fundamentals.md          ← GitHub features, interface, collaboration
├── 03_Git_Commands_CheatSheet.md      ← Every command, organized and searchable
├── 04_Branching_and_Merging.md        ← Branches, merges, rebases, conflicts
├── 05_Forking_and_Pull_Requests.md    ← Open source workflows, PRs, reviews
├── 06_Remotes_and_Origin.md           ← Remotes, fetch/pull/push, SSH vs HTTPS
├── 07_GitHub_Workflow_RealWorld.md    ← Team workflows, professional habits
├── 08_Debugging_and_Recovery.md       ← Fixing mistakes, recovering lost work
├── 09_Advanced_Git.md                 ← Rebase, cherry-pick, bisect, internals
├── 10_Deployment_and_Team_Workflows.md← CI/CD, Vercel, Netlify, releases
└── 11_Git_Glossary.md                 ← Definitions for every Git/GitHub term
```

---

## Core Mental Models (Read These First)

### 1. Git is a snapshot machine, not a diff machine
Every commit is a complete snapshot of all tracked files at a moment in time.  
Git is efficient — it deduplicates unchanged files — but conceptually every commit is a full picture.

### 2. Git has three local zones
```
Working Directory  →  Staging Area (Index)  →  Repository (.git)
  (your files)         (what will commit)        (commit history)
```
Commands move content between zones. Most confusion comes from not knowing which zone you're in.

### 3. Branches are just pointers to commits
A branch is a lightweight movable label. When you commit, the label moves forward.  
Nothing is copied. Branching in Git is near-free.

### 4. HEAD is where you are right now
HEAD points to the current branch (or commit). It moves when you commit or switch branches.

### 5. Remote repositories are just other copies
`origin` is a nickname for a URL. `git push` sends your commits there.  
`git pull` is `git fetch` + `git merge`. They are separate operations with separate effects.

---

## Quick-Start Workflows

### Solo project from scratch
```bash
git init
git add .
git commit -m "Initial commit"
gh repo create my-project --public --source=. --push
```

### Clone and start working
```bash
git clone https://github.com/user/repo.git
cd repo
git checkout -b feature/my-feature
# ... make changes ...
git add .
git commit -m "feat: add my feature"
git push -u origin feature/my-feature
```

### Contribute to open source
```bash
# Fork on GitHub first, then:
git clone https://github.com/YOUR_USERNAME/repo.git
git remote add upstream https://github.com/ORIGINAL/repo.git
git checkout -b fix/my-fix
# ... make changes ...
git push origin fix/my-fix
# Open Pull Request on GitHub
```

### Recover from a mistake
```bash
git reflog                        # Find the commit you want to return to
git reset --hard HEAD~1           # Undo last commit (destructive)
git revert HEAD                   # Undo last commit (safe, keeps history)
git stash pop                     # Restore stashed work
```

---

## Why Git Became the Industry Standard

- **Distributed**: every developer has the full history locally
- **Fast**: almost all operations are local, no server roundtrip needed
- **Reliable**: cryptographic hashing means corruption is detectable
- **Branching is free**: unlike SVN, branches are trivially cheap
- **GitHub network effect**: open source moved to GitHub, companies followed
- **Ecosystem**: CI/CD tools, deployment platforms, IDEs all integrate with Git natively

---

## Professional Habits (Summary)

- Commit small and often. One logical change per commit.
- Write commit messages that explain WHY, not just WHAT.
- Never force-push to `main` or shared branches.
- Always pull before starting new work.
- Use branches for every feature or fix, no matter how small.
- Use `.gitignore` from day one.
- Review your `git diff` before every commit.
- Use `git stash` when you need to switch context mid-work.
- Use `git reflog` before panicking — almost nothing is truly lost.

---

*Last updated: 2026 | Reference system for engineers by engineers*
