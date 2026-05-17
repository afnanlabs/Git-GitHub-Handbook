# 02 — GitHub Fundamentals

> GitHub is not Git. Git is the version control system. GitHub is a hosting platform built on top of Git that adds collaboration, project management, CI/CD, and social features.

---

## Table of Contents

1. [GitHub vs Git](#github-vs-git)
2. [Repositories on GitHub](#repositories-on-github)
3. [Public vs Private Repositories](#public-vs-private)
4. [The GitHub Interface — Key Areas](#github-interface)
5. [Issues](#issues)
6. [Pull Requests](#pull-requests)
7. [Forks](#forks)
8. [Stars and Watchers](#stars-and-watchers)
9. [Releases and Tags](#releases-and-tags)
10. [GitHub Discussions](#github-discussions)
11. [GitHub Pages](#github-pages)
12. [GitHub Actions — Overview](#github-actions)
13. [GitHub Contribution Graph](#contribution-graph)
14. [GitHub CLI](#github-cli)
15. [GitHub Security Features](#security-features)
16. [Professional GitHub Habits](#professional-habits)

---

## GitHub vs Git

| Git                              | GitHub                                                                    |
| -------------------------------- | ------------------------------------------------------------------------- |
| Version control system           | Hosting platform                                                          |
| Runs locally on your machine     | Cloud service                                                             |
| Created by Linus Torvalds (2005) | Created by Tom Preston-Werner et al. (2008), acquired by Microsoft (2018) |
| Open source tool                 | Commercial product (free tier + paid plans)                               |
| Core: commits, branches, merges  | Adds: PRs, Issues, Actions, Pages, Projects                               |
| Works without internet           | Requires internet                                                         |

**Alternatives to GitHub**: GitLab (self-hostable), Bitbucket (Atlassian ecosystem), Gitea (self-hosted), Azure DevOps.

GitHub dominates open source and is the industry default. Understanding GitHub is understanding the ecosystem.

---

## Repositories on GitHub

A GitHub repository is a Git repository hosted on GitHub's servers with a web interface layered on top.

### Creating a repository

**Via web:**

1. Click "New" on github.com
2. Name it, choose public/private, optionally initialize with README

**Via GitHub CLI (preferred for developers):**

```bash
gh repo create my-project --public --source=. --remote=origin --push
# Creates the remote repo, sets origin, pushes existing local repo
```

**Via git + manual setup:**

```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/username/repo.git
git push -u origin main
```

### Repository structure on GitHub

```
github.com/username/repo-name
├── Code tab          ← File browser, README display
├── Issues            ← Bug reports, feature requests
├── Pull Requests     ← Code review and merging
├── Actions           ← CI/CD pipelines
├── Projects          ← Kanban/project management
├── Wiki              ← Documentation pages
├── Security          ← Vulnerability alerts, secret scanning
├── Insights          ← Traffic, contributors, dependency graph
└── Settings          ← Repo config, branch protection, access control
```

---

## Public vs Private Repositories

|               | Public                 | Private                            |
| ------------- | ---------------------- | ---------------------------------- |
| Visible to    | Everyone               | Only invited collaborators         |
| Common use    | Open source, portfolio | Client work, proprietary code      |
| Free tier     | Yes                    | Yes (with limits on team features) |
| Can be forked | Yes, by anyone         | Only by collaborators              |
| Searchable    | Yes                    | No                                 |

### When to use private vs public

- **Public**: OSS libraries, personal portfolio projects, educational content
- **Private**: Client projects, internal tools, anything with business logic or secrets

> **⚠️ Warning**: Never commit secrets (API keys, passwords, tokens) to any repository — public OR private. Once pushed, assume it's compromised. Use environment variables and `.gitignore`. GitHub has secret scanning that will alert you automatically.

---

## The GitHub Interface — Key Areas

### Code tab

- File browser: navigate your repo's file tree
- Commit history: per-file or whole-repo
- Branch selector: switch between branches/tags
- README.md is rendered automatically at root
- Green "Code" button: clone URL (HTTPS or SSH)

### README.md — Your project's front page

Every serious repository has a README. It renders as Markdown on the Code tab.

Good README structure:

```markdown
# Project Name

Short description.

## Installation

## Usage

## Contributing

## License
```

---

## Issues

Issues are GitHub's built-in task/bug/discussion system.

### What issues are used for

- Bug reports from users or team members
- Feature requests
- Questions and support (smaller projects)
- Tasks and to-dos linked to code changes

### Issue anatomy

```
Title: [BUG] Login fails with OAuth providers
Body: Steps to reproduce, expected behavior, actual behavior
Labels: bug, priority:high, needs-triage
Assignees: @developer-name
Milestone: v2.1
Projects: Sprint 4
Linked PRs: #145
```

### Labels

Labels categorize issues. Common defaults:

- `bug` — something is broken
- `enhancement` — feature request
- `documentation` — docs work needed
- `good first issue` — appropriate for new contributors
- `help wanted` — maintainer wants community help

### Referencing issues in commits

```bash
git commit -m "fix: resolve login error, closes #42"
# When this commit merges to default branch, GitHub auto-closes issue #42
```

Keywords: `closes`, `fixes`, `resolves` — all auto-close the referenced issue on merge.

### Issue templates

Large projects use `.github/ISSUE_TEMPLATE/` to enforce structured reports:

```
.github/
└── ISSUE_TEMPLATE/
    ├── bug_report.md
    └── feature_request.md
```

---

## Pull Requests

Pull Requests (PRs) are GitHub's mechanism for proposing, reviewing, and merging code changes.

### What a PR is

A PR says: "I made changes on branch X, please review and merge them into branch Y."

It is **not** a Git feature — it's a GitHub feature layered on top of Git branches and merges.

### PR lifecycle

```
1. Create feature branch locally
2. Push branch to GitHub
3. Open PR on GitHub (base: main ← compare: feature-branch)
4. Reviewers are notified
5. Discussion, comments, requested changes
6. Developer pushes fixes to same branch (PR auto-updates)
7. Reviewers approve
8. PR is merged (merge commit / squash / rebase)
9. Feature branch is deleted
10. Issue is auto-closed if referenced
```

### PR merge strategies (GitHub offers all three)

| Strategy             | What it does                                  | When to use                             |
| -------------------- | --------------------------------------------- | --------------------------------------- |
| **Merge commit**     | Adds a merge commit preserving branch history | Team preference, preserves full context |
| **Squash and merge** | Collapses all branch commits into one commit  | Clean main history, feature work messy  |
| **Rebase and merge** | Replays commits onto main, no merge commit    | Linear history advocates                |

### Draft PRs

Open a PR as "Draft" to signal it's not ready for review:

```
[DRAFT] feat: add payment integration
```

Used to get early feedback or to trigger CI before the work is complete.

### Review features

- **Line comments**: comment on specific lines or ranges
- **Suggestions**: reviewers can propose exact code changes, author can accept with one click
- **Review status**: Comment / Approve / Request Changes
- **Required reviewers**: configurable via branch protection rules
- **CODEOWNERS**: `.github/CODEOWNERS` file auto-assigns reviewers based on file paths

```
# .github/CODEOWNERS
*.js @frontend-team
/api/ @backend-team
/docs/ @tech-writers
```

---

## Forks

A fork is a **copy of a repository under your own GitHub account**.

### Why forks exist

- You don't have write access to someone else's repo
- You want to propose changes to an open source project
- You want your own version of a project to diverge independently

### Fork vs Branch

|           | Fork                   | Branch                      |
| --------- | ---------------------- | --------------------------- |
| Lives in  | Your GitHub account    | Same repo                   |
| Use case  | External contributor   | Team member                 |
| PR target | Original repo's branch | Another branch in same repo |

### Fork workflow

```bash
# 1. Fork on GitHub (click Fork button)
# 2. Clone YOUR fork
git clone https://github.com/YOUR_USERNAME/project.git

# 3. Add original as upstream
git remote add upstream https://github.com/ORIGINAL/project.git

# 4. Work on a branch in your fork
git checkout -b fix/typo-in-readme

# 5. Push to your fork
git push origin fix/typo-in-readme

# 6. Open PR from your fork's branch → original repo's main
```

See `05_Forking_and_Pull_Requests.md` for the complete fork workflow.

---

## Stars and Watchers

### Stars

- Bookmarking mechanism (like a "favorite")
- Also a social signal — high star count = popular/trusted project
- Your starred repos are visible at `github.com/stars`
- Use them to save projects for later reference

### Watchers

- Subscribe to all notifications from a repo (issues, PRs, releases, comments)
- More granular than starring
- Options: "Not watching" / "Watching" / "Releases only" / "Ignoring"

### GitHub Notifications

Manage at `github.com/notifications`. Notified when:

- Someone mentions you (`@username`)
- Someone comments on your issue/PR
- CI fails on your PR
- Watched repo has activity
- Team is mentioned

> **Tip**: Configure notification settings carefully. Default "watching" all repos you join will flood your inbox. Use "Releases only" for most repos.

---

## Releases and Tags

### Tags

A tag is a named pointer to a specific commit. Used to mark versions.

```bash
git tag v1.0.0                      # Lightweight tag
git tag -a v1.0.0 -m "Version 1.0" # Annotated tag (preferred)
git push origin v1.0.0              # Push single tag
git push origin --tags              # Push all tags
```

### GitHub Releases

Built on top of Git tags, but adds:

- Release notes (markdown)
- Binary attachments (compiled executables, build artifacts)
- Pre-release flag
- "Latest release" API endpoint

Creating a release:

1. Go to repo → Releases → "Draft a new release"
2. Create or select a tag (e.g., `v2.0.0`)
3. Write release notes (GitHub can auto-generate from merged PRs)
4. Publish

**Semantic Versioning** (`MAJOR.MINOR.PATCH`):

- `MAJOR`: breaking changes
- `MINOR`: new features, backward-compatible
- `PATCH`: bug fixes, backward-compatible

---

## GitHub Discussions

A forum-like space within a repository, separate from Issues.

- Issues: specific bugs and tasks (closeable, assignable)
- Discussions: open-ended Q&A, announcements, ideas, show-and-tell

Common categories:

- 📣 Announcements
- 💬 General
- 💡 Ideas
- 🙏 Q&A
- 🙌 Show and Tell

Used by large open source projects (React, TypeScript, etc.) to reduce issue noise.

---

## GitHub Pages

Free static site hosting directly from a GitHub repository.

### How it works

- GitHub serves files from a specific branch or `/docs` folder
- Supports custom domains
- Automatically runs Jekyll (static site generator) if detected
- Or serve raw HTML/CSS/JS

### Setup

1. Go to repo → Settings → Pages
2. Choose branch and folder (root or `/docs`)
3. Save — your site is live at `username.github.io/repo-name`

### Use cases

- Project documentation
- Personal portfolio (`username.github.io`)
- Static websites for open source libraries

### With other static site generators

```yaml
# .github/workflows/deploy.yml
# Build with Vite/Next.js/Astro, then deploy to Pages
```

---

## GitHub Actions — Overview

GitHub Actions is a CI/CD and automation platform built into GitHub.

### Core concepts

- **Workflow**: A YAML file in `.github/workflows/` that defines automation
- **Trigger (on)**: What causes the workflow to run (push, PR, schedule, manual)
- **Job**: A group of steps that run on the same machine
- **Step**: A single command or Action
- **Action**: A reusable unit of work (GitHub Marketplace has thousands)
- **Runner**: The machine that runs your job (GitHub-hosted or self-hosted)

### Basic workflow example

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4 # Check out your code
      - uses: actions/setup-node@v4 # Install Node.js
        with:
          node-version: "20"
      - run: npm ci # Install dependencies
      - run: npm test # Run tests
      - run: npm run build # Build
```

### Common use cases

- Run tests on every PR (block merging if tests fail)
- Lint and type-check code
- Auto-deploy to Vercel, Netlify, AWS on push to main
- Send Slack notifications
- Publish npm packages on release
- Auto-label PRs, assign reviewers

See `10_Deployment_and_Team_Workflows.md` for detailed Actions patterns.

---

## Contribution Graph

The green grid on your GitHub profile — shows your commit/PR/review/issue activity over the past year.

### What counts as a contribution

- Commits to default branch or gh-pages
- Opening issues
- Opening pull requests
- Submitting pull request reviews
- Commits to a fork that are merged upstream

### What does NOT count

- Commits to branches that aren't the default branch
- Commits to forks where you haven't opened a PR
- Commits before account ownership (transferred repos)
- Commits using an email not associated with your GitHub account

> **Common issue**: Your commits don't appear in the graph.  
> Fix: ensure your Git `user.email` matches your GitHub primary/verified email.
>
> ```bash
> git config --global user.email "your@github-email.com"
> ```

---

## GitHub CLI

`gh` is GitHub's official CLI tool. Dramatically speeds up GitHub workflow.

```bash
# Install: https://cli.github.com
gh auth login                          # Authenticate

gh repo create                         # Create repo
gh repo clone owner/repo               # Clone repo

gh pr create                           # Create PR interactively
gh pr list                             # List open PRs
gh pr checkout 123                     # Check out PR #123 locally
gh pr merge 123                        # Merge PR
gh pr review 123 --approve             # Approve PR

gh issue create                        # Create issue
gh issue list                          # List issues
gh issue close 42                      # Close issue

gh run list                            # List Actions runs
gh run watch                           # Watch a run in progress

gh release create v1.0.0               # Create release
```

---

## Security Features

### Dependabot

Automatically opens PRs to update vulnerable dependencies. Configure in `.github/dependabot.yml`.

### Secret scanning

GitHub scans for accidentally committed secrets (API keys, tokens). Alerts you and may notify the secret provider.

### Code scanning (CodeQL)

Static analysis to find security vulnerabilities in your code. Integrate via GitHub Actions.

### Branch protection rules

Under repo Settings → Branches:

- Require PR before merging
- Require status checks (CI must pass)
- Require signed commits
- Prevent force pushes to main
- Require linear history

```
Recommended protection for main:
✓ Require a pull request before merging
  ✓ Require approvals: 1
  ✓ Dismiss stale reviews on new push
✓ Require status checks to pass
  ✓ Require branches to be up to date
✓ Do not allow bypassing the above settings
```

---

## Professional GitHub Habits

- **Write meaningful PR descriptions** — describe what changed and why, link issues
- **Keep PRs small** — under 400 lines of change is reviewable; 2000 lines is a nightmare
- **Use draft PRs** for work in progress
- **Respond to review comments promptly** — don't let PRs stagnate
- **Resolve conversations** after addressing a review comment
- **Use `CODEOWNERS`** to auto-assign the right reviewers
- **Set up branch protection** on `main` — protect your production branch
- **Configure Dependabot** — automate dependency security updates
- **Never commit secrets** — use GitHub Secrets for Actions, environment variables for apps
- **Archive repos** instead of deleting — preserves history and links

---

_Next: `03_Git_Commands_CheatSheet.md` — Every important Git command, organized and searchable_
