# 10 — Deployment and Team Workflows

> How Git integrates with CI/CD pipelines, deployment platforms, release workflows, and professional engineering team operations.

---

## Table of Contents

1. [Git and Deployment — The Core Idea](#git-and-deployment)
2. [Branch-Based Deployment Model](#branch-based-deployment)
3. [GitHub Actions — Practical CI/CD](#github-actions-practical)
4. [Deploying with Vercel](#deploying-with-vercel)
5. [Deploying with Netlify](#deploying-with-netlify)
6. [Deploying with Heroku](#deploying-with-heroku)
7. [Deploying with AWS / Docker](#deploying-with-aws-docker)
8. [Environment Strategy](#environment-strategy)
9. [Release Workflows](#release-workflows)
10. [Semantic Versioning and Automated Releases](#semantic-versioning)
11. [GitHub Environments and Secrets](#github-environments-secrets)
12. [Production Hotfix Workflow](#production-hotfix-workflow)
13. [Team Onboarding with Git](#team-onboarding)
14. [Git in a Monorepo](#git-in-a-monorepo)
15. [What a Mature CI/CD Pipeline Looks Like](#mature-cicd-pipeline)

---

## Git and Deployment — The Core Idea

Modern deployment is **triggered by Git events**. You don't manually upload files. You push a commit, and automation takes over.

The core model:
```
developer pushes to branch
         ↓
CI runs (tests, lint, build)
         ↓
if CI passes and branch = main
         ↓
CD deploys to production
```

This is **Continuous Integration** (CI) + **Continuous Deployment** (CD). Git is the trigger for the entire pipeline.

---

## Branch-Based Deployment Model

Different branches map to different environments:

```
feature/* branches  → Preview deployments (per-PR)
main (or develop)   → Staging environment
production branch   → Production environment
```

### Common environment-branch mappings

| Branch | Environment | Who accesses it |
|---|---|---|
| `feature/xxx` | Preview URL (per-PR) | Developer + reviewer |
| `develop` or `main` | Staging | QA team, internal |
| `release/x.x` | Pre-production | Final QA, stakeholders |
| `production` | Production | End users |

### Two-branch production model (common in startups)

```
main        → auto-deploys to staging
production  → auto-deploys to production (merge main here when ready)
```

Deployment to production = a PR from `main` → `production`. Clean, auditable, reversible (revert the merge).

### Single-branch model (GitHub Flow + continuous deployment)

```
main → immediately deploys to production on every merge
```

Simple. Works when your test suite and CI are trustworthy. Used by many high-confidence SaaS teams.

---

## GitHub Actions — Practical CI/CD

### Key concepts
```
.github/workflows/ci.yml    ← One workflow file per job category
```

A workflow has:
- **`on`**: triggers (push, pull_request, schedule, workflow_dispatch)
- **`jobs`**: parallel or sequential groups of steps
- **`steps`**: individual commands or reusable Actions
- **`env`**: environment variables
- **`secrets`**: encrypted values from GitHub Settings

### CI Workflow: Test on every PR and push

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    name: Test & Lint
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18, 20]   # Test on multiple Node versions

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run type check
        run: npm run typecheck

      - name: Run tests
        run: npm test -- --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
```

### CD Workflow: Deploy to production on merge to main

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: []   # Could depend on a test job

    environment:
      name: production
      url: https://myapp.com

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build
        env:
          VITE_API_URL: ${{ secrets.PRODUCTION_API_URL }}

      - name: Deploy to Vercel
        run: npx vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
```

### Workflow for PR preview deployments

```yaml
# .github/workflows/preview.yml
name: Preview Deployment

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy Preview
        id: deploy
        run: |
          URL=$(npx vercel --token=${{ secrets.VERCEL_TOKEN }})
          echo "preview_url=$URL" >> $GITHUB_OUTPUT

      - name: Comment PR with preview URL
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '🚀 Preview deployed: ${{ steps.deploy.outputs.preview_url }}'
            })
```

### Caching dependencies for speed

```yaml
- name: Cache node_modules
  uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

### Conditional steps

```yaml
- name: Deploy to production
  if: github.ref == 'refs/heads/main' && github.event_name == 'push'
  run: ./deploy.sh

- name: Notify Slack on failure
  if: failure()
  uses: slackapi/slack-github-action@v1
  with:
    payload: '{"text":"❌ Build failed on ${{ github.ref }}"}'
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### Matrix builds

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node: [18, 20]
    exclude:
      - os: windows-latest
        node: 18
```

Runs all combinations in parallel. Great for cross-platform libraries.

---

## Deploying with Vercel

Vercel is the simplest Git-integrated deployment platform, designed primarily for Next.js and frontend apps.

### Setup
1. Connect GitHub account at vercel.com
2. Import your repository
3. Configure build command and output directory
4. Deploy

That's it. Every push to `main` deploys. Every PR gets a preview URL.

### How Vercel uses Git
- **Main branch push** → Production deployment at `yourapp.vercel.app` or custom domain
- **Any other branch push** → Preview deployment at `yourapp-git-branch-name.vercel.app`
- **PR opened** → Preview URL posted as PR comment

### Vercel config (optional fine control)

```json
// vercel.json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "installCommand": "npm ci",
  "framework": "vite",
  "env": {
    "NODE_ENV": "production"
  },
  "routes": [
    { "src": "/(.*)", "dest": "/index.html" }
  ]
}
```

### Deploying via CLI in GitHub Actions

```bash
npm install -g vercel

# Staging
vercel --token=$VERCEL_TOKEN

# Production
vercel --prod --token=$VERCEL_TOKEN
```

### Environment variables in Vercel
Set in Vercel dashboard → Project → Settings → Environment Variables.  
Scope them per environment: Production, Preview, Development.

Never commit `.env` files — let Vercel inject them at build time.

---

## Deploying with Netlify

Similar to Vercel but with stronger support for static sites, serverless functions, and edge functions.

### Git integration
- Connect repo in Netlify dashboard
- Every push to configured branch → production deploy
- PRs get deploy previews automatically

### Netlify config

```toml
# netlify.toml
[build]
  command = "npm run build"
  publish = "dist"

[build.environment]
  NODE_VERSION = "20"

# Branch deploy settings
[[context.staging.environment]]
  API_URL = "https://staging-api.example.com"

[[context.production.environment]]
  API_URL = "https://api.example.com"

# Redirect all routes to index.html (SPA support)
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

### Deploy via CLI

```bash
npm install -g netlify-cli
netlify login

netlify deploy --dir=dist              # Draft deploy
netlify deploy --dir=dist --prod       # Production deploy
```

### Netlify-specific features
- **Split testing**: Route % of traffic to different branches
- **Rollback**: One-click previous deploy restoration
- **Deploy lock**: Prevent new deploys while investigating production issue

---

## Deploying with Heroku

For full-stack apps with backend servers (Node, Python, Ruby, etc.).

### Setup

```bash
heroku login
heroku create myapp-name
# Automatically adds heroku remote: git@heroku.com:myapp-name.git

git push heroku main    # Deploy
```

### Procfile
Tells Heroku how to start your app:
```
web: node server.js
worker: node worker.js
```

### Heroku with GitHub Actions

```yaml
- name: Deploy to Heroku
  uses: akhileshns/heroku-deploy@v3.13.15
  with:
    heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
    heroku_app_name: myapp-name
    heroku_email: you@example.com
```

### Heroku review apps
Heroku can auto-create a temporary app for each PR — similar to Vercel/Netlify previews.

Configure in `app.json`:
```json
{
  "name": "My App",
  "scripts": {
    "postdeploy": "npm run db:migrate"
  },
  "env": {
    "NODE_ENV": {
      "value": "staging"
    }
  }
}
```

---

## Deploying with AWS / Docker

For teams running their own infrastructure.

### Docker + GitHub Actions → AWS ECS

```yaml
# .github/workflows/deploy-aws.yml
name: Deploy to AWS

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build, tag, and push image to ECR
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: myapp
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG

      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster myapp-cluster \
            --service myapp-service \
            --force-new-deployment
```

### Tagging Docker images by Git commit or tag

```bash
# Use the Git SHA as the image tag
docker build -t myapp:$(git rev-parse --short HEAD) .

# Or use the Git tag for releases
docker build -t myapp:$(git describe --tags) .
```

This creates a direct link between a running container and a specific Git commit — critical for debugging production issues.

---

## Environment Strategy

### What environments are for

| Environment | Purpose | Who uses it | Deployed on |
|---|---|---|---|
| Local | Developer testing | Individual dev | Developer's machine |
| Preview | PR-specific testing | Dev + reviewer | Per-PR auto-deploy |
| Development | Integration testing | Dev team | Push to `develop` |
| Staging | Pre-production QA | QA team + stakeholders | Merge to `staging` |
| Production | Live system | End users | Merge to `main` or `production` |

### Environment variables per environment

Never hard-code environment-specific config. Use environment variables.

```bash
# .env.development
API_URL=http://localhost:3001
DEBUG=true

# .env.staging
API_URL=https://staging-api.myapp.com
DEBUG=false

# .env.production
API_URL=https://api.myapp.com
DEBUG=false
```

```bash
# .gitignore — CRITICAL
.env
.env.local
.env.*.local
```

`.env.example` (committed) documents required variables without exposing values:
```bash
# .env.example
API_URL=
DATABASE_URL=
JWT_SECRET=
STRIPE_API_KEY=
```

---

## Release Workflows

### Simple release with tags

```bash
# Ensure main is clean and tested
git switch main
git pull origin main

# Create annotated tag
git tag -a v2.1.0 -m "Release v2.1.0: add payment integration"

# Push tag to GitHub
git push origin v2.1.0

# GitHub Actions can trigger on tag push
# Create GitHub Release (manually or via CLI)
gh release create v2.1.0 --generate-notes
```

### GitHub Actions on tag push

```yaml
on:
  push:
    tags:
      - 'v*'    # Trigger on any tag like v1.0.0, v2.1.3

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build
        run: npm run build

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          files: dist/**
          generate_release_notes: true
```

### Release branch workflow

For projects that need a dedicated QA period before releasing:

```bash
# Cut a release branch from develop
git switch develop
git pull origin develop
git switch -c release/v2.1.0

# Bump version number
npm version minor   # Updates package.json, creates a tag
git push origin release/v2.1.0

# QA runs on this branch
# Only bug fixes go in here:
git switch -c fix/payment-edge-case release/v2.1.0
# ... fix ...
git switch release/v2.1.0
git merge fix/payment-edge-case

# When QA passes: merge to main AND back to develop
git switch main
git merge --no-ff release/v2.1.0
git tag -a v2.1.0 -m "Release v2.1.0"
git push origin main --tags

git switch develop
git merge --no-ff release/v2.1.0
git push origin develop

# Delete release branch
git branch -d release/v2.1.0
git push origin --delete release/v2.1.0
```

---

## Semantic Versioning and Automated Releases

### semantic-release — Fully automated version management

`semantic-release` reads commit messages (Conventional Commits format) and:
- Determines the next version number automatically
- Generates a CHANGELOG
- Creates a GitHub Release
- Publishes to npm (optional)

```bash
npm install --save-dev semantic-release \
  @semantic-release/changelog \
  @semantic-release/git \
  @semantic-release/github
```

```json
// .releaserc.json
{
  "branches": ["main"],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/changelog",
    "@semantic-release/github",
    "@semantic-release/git"
  ]
}
```

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0   # Need full history for semantic-release
          persist-credentials: false
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npx semantic-release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

How version bumps work with conventional commits:
- `fix:` commit → **PATCH** bump (1.0.0 → 1.0.1)
- `feat:` commit → **MINOR** bump (1.0.0 → 1.1.0)
- `feat!:` or `BREAKING CHANGE:` → **MAJOR** bump (1.0.0 → 2.0.0)

---

## GitHub Environments and Secrets

### Secrets
Encrypted values available to GitHub Actions workflows.

```yaml
# Add secrets in: repo Settings → Secrets and variables → Actions

# Use in workflow:
env:
  DATABASE_URL: ${{ secrets.DATABASE_URL }}
  API_KEY: ${{ secrets.PRODUCTION_API_KEY }}
```

Secret types:
- **Repository secrets**: available to all workflows in the repo
- **Environment secrets**: available only when deploying to that environment
- **Organization secrets**: shared across org repos

### Environments

Environments add deployment protection rules:

```yaml
# Settings → Environments → production
# Protection rules:
#   ✓ Required reviewers: @senior-dev, @tech-lead
#   ✓ Wait timer: 5 minutes
#   ✓ Deployment branches: main only
```

```yaml
# Reference in workflow:
jobs:
  deploy-prod:
    environment:
      name: production
      url: https://myapp.com
    # This job will pause and require approval from required reviewers
```

### Using environment-specific secrets

```yaml
jobs:
  deploy-staging:
    environment: staging
    steps:
      - run: deploy.sh
        env:
          DB_URL: ${{ secrets.DB_URL }}   # staging environment's version of this secret

  deploy-production:
    environment: production
    steps:
      - run: deploy.sh
        env:
          DB_URL: ${{ secrets.DB_URL }}   # production environment's version of this secret
```

---

## Production Hotfix Workflow

When production is broken and you need to fix it NOW:

```bash
# Step 1: Branch from production (or main if single-branch)
git switch main
git pull origin main
git switch -c hotfix/payment-500-error

# Step 2: Fix it
# (make minimal change — this is not the time to refactor)
git add src/payment/processor.js
git commit -m "fix: handle null response from payment gateway"

# Step 3: Test quickly (CI runs on push)
git push -u origin hotfix/payment-500-error

# Step 4: Open PR, get expedited review (flag as urgent)
gh pr create --title "HOTFIX: payment gateway null response crash" \
             --label "hotfix,urgent" \
             --reviewer "@on-call-dev"

# Step 5: Merge (squash or merge commit)
# CI green + 1 approval (relaxed review standards for hotfixes)

# Step 6: Verify deployment and monitor
# Check dashboards, error tracking (Sentry), logs

# Step 7: Also merge hotfix to develop (if using Git Flow)
git switch develop
git merge main   # or cherry-pick the hotfix commit

# Step 8: Post-incident review (blameless)
# Create issue: root cause, follow-up improvements, timeline
```

### Post-hotfix habits
- Write a post-mortem issue in GitHub
- Add a regression test for the bug
- Check if the same bug exists elsewhere in the codebase

---

## Team Onboarding with Git

### Onboarding checklist for new developers

```markdown
## Git Setup
- [ ] Install Git (git --version)
- [ ] Set global user.name and user.email
- [ ] Generate SSH key and add to GitHub
- [ ] Configure default editor (git config --global core.editor)
- [ ] Set pull.rebase true (git config --global pull.rebase true)
- [ ] Install GitHub CLI (gh auth login)

## Project Setup
- [ ] Clone the repository
- [ ] Read CONTRIBUTING.md
- [ ] Read .github/PULL_REQUEST_TEMPLATE.md
- [ ] Understand the branch naming convention
- [ ] Understand the commit message convention
- [ ] Run the project locally

## First Contribution
- [ ] Find a "good first issue" or ask for one
- [ ] Create a feature branch
- [ ] Make the change, open PR
- [ ] Get review, address feedback
- [ ] PR merged ✓
```

### Recommended `.gitconfig` for team members

```ini
[user]
    name = Developer Name
    email = dev@company.com

[core]
    editor = code --wait
    autocrlf = input          # Mac/Linux: input, Windows: true

[pull]
    rebase = true

[push]
    default = current

[merge]
    tool = vscode

[alias]
    s = status -sb
    lg = log --oneline --graph --all --decorate
    undo = reset --soft HEAD~1
    unstage = restore --staged

[init]
    defaultBranch = main
```

---

## Git in a Monorepo

Large organizations (Google, Meta, Airbnb, Nx) use a single repository for all services and libraries.

### Challenges
- `git log` shows changes across all services
- CI must not run ALL tests on every commit
- Large repos are slow to clone

### Solutions

**Sparse checkout** — only check out relevant directories:
```bash
git sparse-checkout init --cone
git sparse-checkout set apps/frontend libs/ui
# Only these directories are in your working directory
```

**Partial clone** — download only recent history:
```bash
git clone --filter=blob:none --depth=50 https://github.com/org/monorepo.git
```

**Path-based CI filtering** — only run relevant jobs:
```yaml
# GitHub Actions: run frontend CI only if frontend files changed
on:
  push:
    paths:
      - 'apps/frontend/**'
      - 'libs/ui/**'
```

**Tools for monorepos**: Nx, Turborepo, Lerna, Bazel — these add caching and task orchestration on top of Git.

---

## What a Mature CI/CD Pipeline Looks Like

```
PR opened
    │
    ├── Lint (ESLint, Prettier) ──── 30s
    ├── Type check (TypeScript) ──── 45s
    ├── Unit tests ────────────────── 2 min
    ├── Integration tests ─────────── 4 min
    ├── Security scan (CodeQL) ────── 3 min (runs in parallel)
    ├── Dependency audit ──────────── 30s
    └── Preview deployment ────────── 2 min
              │
              ▼
    PR comment: "✅ All checks passed. Preview: https://..."

PR approved + merged to main
    │
    ├── Same checks run again ──────── (required)
    ├── Build production bundle ────── 3 min
    ├── Push Docker image to ECR ───── 2 min
    ├── Deploy to staging ──────────── 2 min
    └── Run smoke tests on staging ─── 1 min
              │
              ▼
    [Manual approval required] ← GitHub Environment protection rule

    Approved
    │
    ├── Deploy to production ───────── 3 min
    │   (blue/green or canary rollout)
    ├── Run smoke tests on production ─ 1 min
    └── Notify Slack #deployments ───── instant
              │
              ▼
    Monitor: Sentry, Datadog, PagerDuty
```

### Key metrics for CI/CD health
- **Pipeline duration**: under 10 min for most teams, under 5 min ideal
- **Flaky test rate**: should be near 0% — flaky tests erode trust in CI
- **Deployment frequency**: daily or multiple times per day for high-performing teams
- **Mean time to recovery (MTTR)**: how fast you fix production issues

---

*Next: `11_Git_Glossary.md` — Definitions for every Git and GitHub term*
