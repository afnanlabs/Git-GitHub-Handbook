# 07 — GitHub Workflow: Real-World

> How solo developers, small teams, and large engineering organizations actually use Git and GitHub day to day.

---

## Table of Contents

1. [Solo Developer Workflow](#solo-developer-workflow)
2. [Small Team Workflow (2–10 devs)](#small-team-workflow)
3. [GitHub Flow — The Standard Model](#github-flow)
4. [Git Flow — The Classic Enterprise Model](#git-flow)
5. [Trunk-Based Development](#trunk-based-development)
6. [Open Source Maintainer Workflow](#open-source-maintainer-workflow)
7. [Professional Daily Habits](#professional-daily-habits)
8. [Commit Message Standards](#commit-message-standards)
9. [Branch Strategy Decision Guide](#branch-strategy-decision-guide)
10. [GitHub Projects and Issue Tracking](#github-projects)
11. [Using GitHub Effectively](#using-github-effectively)
12. [What Companies Actually Do](#what-companies-actually-do)

---

## Solo Developer Workflow

Most solo developers use a simplified workflow. The key professional habit: **never commit directly to main on anything important**.

### Minimal solo workflow

```bash
# Start work
git pull origin main                      # Always start fresh

# Feature work
git switch -c feature/dark-mode
# ... code ...
git add -p                                 # Stage carefully
git commit -m "feat: add dark mode toggle"
git push -u origin feature/dark-mode

# Review your own diff on GitHub (optional but valuable)
# Open PR, review the diff, then self-merge

# Or merge locally:
git switch main
git merge --no-ff feature/dark-mode
git push origin main
git branch -d feature/dark-mode
git push origin --delete feature/dark-mode
```

### Why branch even as a solo developer
- GitHub CI runs on branches — PRs trigger your test suite
- You can pause mid-feature and switch to an urgent fix
- Clean history: each feature is a unit
- You can open draft PRs to see GitHub's diff view (better than terminal `git diff`)

### Minimum viable `.gitignore` for solo projects
```
node_modules/
.env
.env.local
dist/
build/
.DS_Store
*.log
.idea/
.vscode/
```

---

## Small Team Workflow

Teams of 2–10 developers using GitHub effectively.

### Setup checklist
- [ ] Default branch is `main`
- [ ] Branch protection on `main`: require PR + at least 1 approval
- [ ] CI (GitHub Actions) configured: run tests on every PR
- [ ] PR merge strategy decided (squash / merge commit) and enforced in settings
- [ ] CODEOWNERS file configured
- [ ] Issue templates created
- [ ] Labels created (bug, enhancement, etc.)

### Daily workflow for each developer

```bash
# Morning: sync
git fetch --all --prune
git switch main
git pull origin main

# Start work
git switch -c fix/23-payment-timeout

# Work session
git add -p
git commit -m "fix: increase payment gateway timeout to 30s"
git push -u origin fix/23-payment-timeout

# Open PR on GitHub:
# - Link to issue #23
# - Tag reviewer
# - CI runs automatically

# While waiting for review, work on another branch:
git switch main
git switch -c feature/24-receipt-email
```

### Review rotation
Small teams often do informal review rotation. Avoid:
- Always having the same person review
- Nobody reviewing junior devs' code
- PRs sitting for more than 1 business day

---

## GitHub Flow

**The most widely used workflow.** Simple, effective, works for continuous delivery.

### Rules
1. Anything in `main` is always deployable
2. Create descriptive branches from `main`
3. Push to that branch constantly (remote backup + CI)
4. Open a PR at any time (draft or ready)
5. Merge to `main` only after review + CI green
6. Deploy immediately after merging to `main`

```
main (always deployable, always deployed)
│
├── feature/user-auth
├── fix/login-bug
└── docs/api-guide
        ↓ (PR + review)
main (updated, immediately deployed)
```

### Diagram

```
time →→→→→→→→→→→→→→→→→→→→→→→→→→→

main:    ─────●──────────────────────●───────────────●────────────────────●──
                   \                /                    \               /
feature:            ●───●───●───●──                      ●───●───●───●──
```

### Why it works
- Simple to understand
- CI/CD is trivial to implement
- No version coordination overhead
- Forces small, frequent releases (good thing)

### When GitHub Flow breaks down
- When you need to support multiple production versions simultaneously
- When releases need explicit versioning and staging
- When regulatory requirements require longer QA cycles

---

## Git Flow

The original enterprise branching model by Vincent Driessen (2010). Heavier but structured for explicit version releases.

### Branch types

| Branch | Long-lived? | Merges into | Branched from |
|---|---|---|---|
| `main` | Yes | — | — |
| `develop` | Yes | `main` | `main` |
| `feature/*` | No | `develop` | `develop` |
| `release/*` | No | `main` + `develop` | `develop` |
| `hotfix/*` | No | `main` + `develop` | `main` |

### The flow

```
main    ────────────────────────────────────●─────────────────●────
                                           /|                /|
                                          / |               / |
develop ────────────────────────────────/──●──────────────/──●────
               \           /           /                 /
feature         ●───●───●─/           /                 /
                                     /                 /
release                             ●───●─────────────/
                                                       \
hotfix                                                  ●───●───●
```

### When Git Flow is appropriate
- Applications with scheduled release cycles (mobile apps, enterprise software)
- Teams that maintain multiple production versions
- When releases go through separate QA/staging before production

### When Git Flow is overkill
- Web applications (always deploy latest)
- Small teams
- Microservices with independent deployment

> **Industry reality**: Git Flow has fallen out of favor for web development. Most modern web teams use GitHub Flow or Trunk-Based Development. Git Flow persists in enterprise software, mobile apps, and orgs with formal release management.

---

## Trunk-Based Development

The model used by Google, Facebook, and Netflix at scale. Everyone commits to `main` (the "trunk") as frequently as possible — multiple times per day.

### Core idea
Long-lived branches cause problems. The solution: eliminate them. Integrate constantly.

### How it works

**Small team (2–5 developers):**
```bash
# Branch lives < 1 day
git switch -c fix/button-color
# 30 minutes of work
git add .
git commit -m "fix: correct button hover color"
git push origin fix/button-color
# Auto-approved PR, auto-merges after CI passes
git switch main && git pull
```

**Larger teams:** Use **feature flags** to ship code that isn't "live" yet:
```javascript
if (featureFlags.newPaymentFlow) {
  return <NewCheckout />;
}
return <OldCheckout />;  // Default until flag is enabled
```

This lets you commit incomplete features to main without showing them to users.

### Pros of trunk-based development
- Continuous integration is forced (you must integrate daily)
- Conflicts are caught immediately (small, frequent)
- `main` always has latest working state
- Fast feedback from CI

### Cons
- Requires mature CI/CD infrastructure
- Requires discipline (no broken commits to main)
- Feature flags add complexity
- Requires strong code review culture (reviews must be fast)

---

## Open Source Maintainer Workflow

### Responsibilities
- Review and merge community PRs
- Manage issues (triage, label, close stale)
- Cut releases (tag + GitHub release)
- Maintain CHANGELOG
- Respond to security reports

### Reviewing an external contributor's PR

```bash
# Using GitHub CLI (fastest)
gh pr checkout 147     # Checks out the PR's branch locally

# Run it, test it
npm test
node src/index.js

# Suggest changes inline on GitHub, or:
git add .
git commit -m "suggestion: improve error message"
git push   # Can push to contributor's branch if they enabled it
```

### Handling a security vulnerability
1. Do NOT discuss publicly (no public issue)
2. Communicate via GitHub's private vulnerability reporting
3. Develop fix on a private branch
4. Coordinate disclosure with reporter
5. Release patch + security advisory simultaneously

### Stale issue management
```yaml
# .github/workflows/stale.yml
name: Mark stale issues
on:
  schedule:
    - cron: '0 0 * * *'   # Run daily
jobs:
  stale:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/stale@v8
        with:
          days-before-stale: 60
          days-before-close: 14
          stale-issue-message: 'This issue has been inactive for 60 days. It will be closed in 14 days unless there is further activity.'
```

---

## Professional Daily Habits

These are the habits that separate junior from senior developer Git usage:

### Start of day
```bash
git fetch --all --prune        # Update knowledge of all remotes
git status                     # Check your working state
git stash list                 # Any forgotten stashes?
```

### Before creating a branch
```bash
git switch main
git pull origin main           # Branch from latest main, always
```

### Before every commit
```bash
git status                     # What's staged?
git diff --staged              # Review exactly what you're committing
```

### Before opening a PR
```bash
git log main..HEAD --oneline   # Review your commits
git diff main...HEAD           # Review all changes vs main
git fetch origin && git rebase origin/main  # Update branch first
```

### Before merging a PR
- Verify CI is green
- Verify all review conversations are resolved
- Verify the PR description accurately describes what's being merged

---

## Commit Message Standards

### Conventional Commits (industry standard)
```
type(scope): description

[optional body]

[optional footer(s)]
```

**Types:**
| Type | Use for |
|---|---|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, whitespace (no logic change) |
| `refactor` | Code restructure without feature change |
| `test` | Adding or fixing tests |
| `chore` | Build process, dependency updates |
| `ci` | CI/CD configuration changes |
| `perf` | Performance improvements |
| `revert` | Reverting a previous commit |

**Examples:**
```
feat(auth): add JWT refresh token support
fix(cart): prevent duplicate items on rapid click
docs: add API authentication examples
refactor(user): extract validation into UserValidator class
feat!: redesign API response format   (! = breaking change)
```

**Footer for breaking changes:**
```
feat!: change authentication endpoint

BREAKING CHANGE: /api/auth/login now returns token in `data.token`
instead of `token` at root level. Update all clients.
```

### Why standardized commit messages matter
- Tools like `semantic-release` use them to auto-generate version numbers
- `git log --grep="feat"` becomes a feature search
- Auto-generated CHANGELOGs become accurate
- PRs and issues are easier to audit

---

## Branch Strategy Decision Guide

```
Question 1: Do you need to support multiple production versions at once?
  YES → Git Flow
  NO  → Continue

Question 2: Is your team > 50 developers with mature CI/CD?
  YES → Trunk-Based Development (with feature flags)
  NO  → Continue

Question 3: Do you deploy continuously (multiple times per day)?
  YES → GitHub Flow (possibly Trunk-Based)
  NO  → GitHub Flow with release tags
```

**Default recommendation**: GitHub Flow works for 90% of teams and projects.

---

## GitHub Projects

GitHub's built-in project management (Kanban/table/roadmap).

### Setup
- Create a Project from the org or repo → Projects tab
- Add columns: Backlog, In Progress, In Review, Done
- Issues and PRs are "items" you can add

### Auto-status updates
Configure automation so:
- PRs in draft → "In Progress"
- PRs opened (not draft) → "In Review"
- PR merged → "Done"
- Issue closed → "Done"

### Integration with Issues and PRs
```
Issue #42 → PR #67 → Auto-close issue → Auto-move card to Done
```

---

## Using GitHub Effectively

### Profile README
Create `YOUR_USERNAME/YOUR_USERNAME` repo with a `README.md`. It displays on your profile. Used for personal branding and showcasing work.

### GitHub Search
```
is:open is:issue label:good-first-issue language:python    # Find beginner issues
repo:facebook/react is:pr author:username                  # Your PRs in a repo
org:vercel is:issue                                        # All issues in an org
```

### Keyboard shortcuts on GitHub
```
.         Open repo in github.dev (web VS Code)
t         Fuzzy file finder
/         Focus search bar
?         Show all shortcuts
```

### GitHub CLI power moves
```bash
gh pr create --fill                # Auto-fill PR from branch name + commits
gh pr checks                       # View CI status
gh issue list --label "bug"        # Filter issues
gh repo view --web                 # Open current repo in browser
```

---

## What Companies Actually Do

### Startups (< 20 engineers)
- GitHub Flow or simplified Git Flow
- 1 required review on PRs
- CI with GitHub Actions
- Deploy main to production automatically or with one-click
- Most discussion happens in PR comments

### Mid-size companies (20–200 engineers)
- GitHub Flow with protected branches
- 2+ required reviews
- CODEOWNERS enforcing team-based reviews
- Staging environment matches `develop` or `main`
- Feature flags for large features
- Release branches for scheduled releases

### Large companies (Google, Meta, Microsoft)
- Often trunk-based development with sophisticated tooling
- Monorepos (single repo for all code) or polyrepos
- Internal CI/CD systems (not just GitHub Actions)
- Automated code review tools (linters, formatters, type checkers run as gatekeepers)
- Multiple environments: dev, staging, canary, production
- Change management for high-risk changes

---

*Next: `08_Debugging_and_Recovery.md` — The most practical section: fixing everything that goes wrong*
