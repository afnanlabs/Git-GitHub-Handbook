# Git Commit Message Manual

> A commit message is not a note to yourself.
> It is communication with your future self and your teammates.

---

# Why Commit Messages Matter

A good commit message should answer:

- What changed?
- Why did it change?
- Which area of the project was affected?

A developer should be able to understand the purpose of a commit without opening every changed file. High-quality commit messages improve code reviews, debugging, onboarding, and project maintenance. Conventional Commits is one of the most widely adopted standards for this purpose.

---

# The Standard Format

```text
type(scope): short description
```

Examples:

```text
feat(auth): add password reset flow
fix(api): handle null user response
docs(readme): update installation guide
refactor(cart): simplify checkout logic
```

---

# Commit Types

## feat

New functionality.

```text
feat(profile): add avatar upload feature
feat(auth): implement forgot password flow
```

---

## fix

Bug fixes.

```text
fix(router): prevent invalid route redirect
fix(navbar): correct mobile menu alignment
```

---

## docs

Documentation only.

```text
docs(readme): add project setup instructions
docs(roadmap): update learning resources
```

---

## refactor

Improve code without changing behavior.

```text
refactor(api): extract validation helper
refactor(form): simplify state management
```

---

## style

Formatting only.

```text
style: format files with prettier
```

---

## test

Tests.

```text
test(auth): add login validation tests
```

---

## chore

Maintenance tasks.

```text
chore: update dependencies
chore: configure eslint
chore: clean repository branches
```

---

## Optional Advanced Types

```text
perf     -> performance improvement
build    -> build system changes
ci       -> GitHub Actions / CI changes
revert   -> revert previous commit
```

Examples:

```text
perf(api): reduce database queries
build(vite): update build configuration
ci(actions): add deployment workflow
revert: remove experimental login flow
```

---

# Golden Rules

## Rule 1: Use Imperative Mood

Always write as if giving a command.

Good:

```text
feat(auth): add login validation
fix(api): handle missing user data
```

Bad:

```text
feat(auth): added login validation
fix(api): fixed missing user data
```

Think:

> "If applied, this commit will..."

This is a long-standing Git convention.

---

## Rule 2: One Commit = One Purpose

Bad:

```text
feat: add login page and update README and fix navbar
```

Good:

```text
feat(auth): add login page

docs(readme): update setup instructions

fix(navbar): correct menu spacing
```

Every commit should represent a single logical change.

---

## Rule 3: Explain Why

Bad:

```text
fix: update validation
```

Good:

```text
fix(auth): prevent empty email submission
```

Future developers care more about the reason than the implementation details.

---

## Rule 4: Avoid Generic Messages

Never write:

```text
update code
changes
final changes
new update
fixed stuff
modified files
```

These messages become useless after a few days.

---

## Rule 5: Keep Subject Short

Aim for:

```text
50 characters or less
```

Examples:

Good:

```text
feat(auth): add password reset flow
```

Too long:

```text
feat(auth): add a complete password reset system with email verification support
```

Use the commit body if additional explanation is needed.

---

# When To Add A Commit Body

For simple changes:

```bash
git commit -m "fix(api): handle null response"
```

For important changes:

```text
feat(auth): add password reset flow

Allow users to reset forgotten passwords
through email verification.

This reduces support requests and aligns
with current authentication requirements.
```

The body should explain:

- Why the change was needed
- Important design decisions
- Side effects

Not every line of code that changed.

---

# Real Examples

## React Component

```text
feat(navbar): add responsive navigation menu
```

---

## React Hook

```text
feat(auth): create useAuth hook
```

---

## Bug Fix

```text
fix(router): prevent redirect loop
```

---

## Styling

```text
style(ui): improve dashboard spacing
```

---

## Refactoring

```text
refactor(form): extract validation logic
```

---

## Documentation

```text
docs(readme): add project setup guide
```

---

# Examples From Your Repository

Instead of:

```text
ADD: new yt list for journey
```

Use:

```text
docs(resources): add React learning playlist
```

---

Instead of:

```text
Update the Best REACT Roadmap
```

Use:

```text
docs(roadmap): expand React learning roadmap
```

---

Instead of:

```text
Added README.md
```

Use:

```text
docs(readme): add project overview
```

---

Instead of:

```text
Update README
```

Use:

```text
docs(readme): add Pull Shark instructions
```

---

# Team-Level Commit Quality

Poor:

```text
fix bug
```

Average:

```text
fix(auth): resolve login validation issue
```

Professional:

```text
fix(auth): prevent login with empty credentials
```

The professional version explains the exact problem.

---

# Personal Commit Workflow

Before every commit ask:

### 1. What changed?

Example:

```text
Added authentication hook
```

### 2. Which area changed?

Example:

```text
auth
```

### 3. Which type is it?

Example:

```text
feat
```

### Final Message

```text
feat(auth): add authentication hook
```

---

# Quick Cheatsheet

```text
FORMAT

type(scope): short description
```

### Types

```text
feat      -> new feature
fix       -> bug fix
docs      -> documentation
refactor  -> code improvement without behavior change
style     -> formatting only
test      -> tests
chore     -> maintenance
perf      -> performance improvement
build     -> build system changes
ci        -> CI/CD changes
revert    -> revert commit
```

### Rules

```text
✔ Use imperative mood
✔ Keep it short
✔ One commit = one purpose
✔ Explain why
✔ Use scope when possible
✔ Be specific

✘ update code
✘ final changes
✘ fixed stuff
✘ modified files
✘ new update
```

### Examples

```text
feat(auth): add forgot password flow
fix(api): handle null user response
docs(readme): update installation guide
refactor(cart): simplify checkout logic
chore(deps): update React Router
test(auth): add login tests
```



### Summary 
```text

Format - 
type(scope): short description
Example:
feat(auth): add login page

This cover 95% of real-world commits
1. feat -> Add a new feature to the codebase
2. fix -> Patch a bug or resolve an issue
3. docs -> Update documentation, markdown files, or code comments
4. refactor -> Rewrite or restructure code without changing its external behavior
5. style -> Modify UI layout, formatting, or code whitespace without functional changes
6. test -> Add, update, or fix test suites and validation cases
7. chore -> Perform routine tasks, maintenance, or build process updates
8. perf -> Optimize code performance, speed, or resource consumption
9. build -> Modify build tools, configurations, or external scripts
10. ci -> Update continuous integration scripts and deployment workflows
11. revert -> Roll back a previous commit or change
12. hotfix -> Apply an urgent patch directly to production code
13. wip -> Save a temporary checkpoint for work currently in progress
14. init -> Create the initial repository structure or first project commit
15. release -> Package, tag, or deploy a specific version of the application
16. merge -> Combine separate branches together
17. security -> Resolve vulnerabilities, patch leaks, or improve token validation
18. deps -> Upgrade, downgrade, or update package dependency versions
19. config -> Modify environment variables, system settings, or API parameters
20. ux -> Enhance the user interface flow, accessibility, or overall user experience
21. ops -> Update server infrastructure, container orchestration, or live operations setups
22. arch -> Restructure structural service layers, system design, or folder hierarchies
23. bump -> Increment version numbers or release tags
24. seed -> Populate the database with initial dummy, lookup, or test data
25. sample -> Add example payload files, mock requests, or mock responses
26. lint -> Fix code styling errors, formatting issues, or linter warnings without changing logic
27. typo -> Fix minor spelling mistakes in code, comments, or UI text
28. i18n -> Add or update internationalization, translations, or language files
29. l10n -> Handle localization changes for specific regions (currencies, date formats)
30. infra -> Modify Infrastructure-as-Code setups like Terraform, AWS configs, or Docker networks (separate from dev ops)
31. vendor -> Update third-party binaries, libraries, or vendored dependencies managed directly in the repo
32. cleanup -> Remove dead code, deleted assets, or unused files from the directory
33. feat! -> Introduce a breaking change with a new feature
34. fix! -> Introduce a breaking change while fixing something
35. BREAKING CHANGE -> Describe incompatible API or behavior changes in the commit body/footer
36. db -> Database schema changes, migrations, or indexes
37. ui -> User interface changes (some teams use this instead of ux)
38. api -> API contract or endpoint changes
39. assets -> Images, fonts, icons, media, and static resources
40. move -> Rename or relocate files and directories
41. remove -> Delete features, modules, or files
42. experiment -> Prototypes, spikes, or proof-of-concepts
43. sync -> Synchronization with upstream repositories or generated content
44. proto -> Update Protocol Buffers, OpenAPI schemas, or generated definitions
45. generated -> Changes to auto-generated files
46. data -> Update datasets or configuration data
47. migration -> Database or framework migrations
48.rollback -> Emergency rollback separate from a normal revert

```
