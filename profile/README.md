# Contributing to Nexus PLM — Branching & Workflow Guide

This document describes how all code changes must flow through the Nexus-PLM repositories. Read it before opening your first pull request.

---

## The Three-Tier Model

```
feature/<n>-<slug>  ──PR──►  next  ──auto-PR──►  main
```

| Branch | Role | Protected? |
|--------|------|-----------|
| `main` | Production — what is deployed/released | ✅ Yes |
| `next` | Integration / UAT — what is being prepared for release | ✅ Yes |
| `feature/*` | Your working branch | ❌ No |

**Rules:**
- `feature/*` branches are created from `next`, not from `main`
- Changes reach `next` via a pull request from a `feature/*` branch
- Changes reach `main` via a pull request from `next` only — never directly
- Direct push to `main` or `next` is blocked for everyone, including org admins

---

## Step-by-Step: Contributing a Change

### 1. Find or create an issue

All work should be tracked on the [Nexus Issue Tracker](https://github.com/orgs/Nexus-PLM/projects/1). Open an issue in the relevant repository before starting work, or pick up an existing open issue.

### 2. Create your feature branch

Use the branch creation script — it names your branch correctly and points it at `next`:

```powershell
cd C:\GitHub\Nexus.PLM.Scripts\Branching-Strategy
.\05-create-branch-from-issue.ps1
# Lists all open issues in the project board, then:
.\05-create-branch-from-issue.ps1 -IssueUrl https://github.com/Nexus-PLM/<repo>/issues/<n>
```

Branch naming convention: `feature/<issue-number>-<title-slug>`

Examples:
```
feature/8-repo-refactoring
feature/12-fix-login-button-alignment
feature/34-add-sqlite-migration-support
```

To switch to the branch locally after creating it:
```bash
cd C:\GitHub\<repo>
git fetch
git checkout feature/<n>-<slug>
```

> **Do not** create a working branch from `main`. If you do, it will be automatically deleted with an explanation comment. Always branch from `next`.

### 3. Make your changes

Work on your `feature/*` branch as normal. Commit as often as you like — only the PR merge matters for the shared branches.

```bash
git add .
git commit -m "your message"
git push origin feature/<n>-<slug>
```

### 4. Open a pull request to `next`

Open a PR from your `feature/*` branch targeting **`next`** (not `main`).

When the PR is opened:
- It is automatically added as a card on the [Nexus Issue Tracker](https://github.com/orgs/Nexus-PLM/projects/1) project board
- CI runs against the `next` branch trigger

If your PR description contains `Closes #<n>` or `Fixes #<n>`, the linked issue will be automatically closed when the PR merges.

> **Do not** open a PR directly to `main`. If you do, it will be automatically closed with an explanation comment.

### 5. Get your PR reviewed and merged

Once your `feature/*` PR is approved and merged into `next`, your changes are in the integration branch.

### 6. Promotion to main (automatic)

Merging anything into `next` automatically triggers a pull request from `next` to `main`. You do not open this PR — it appears on its own within seconds.

```
Your PR merges to next  →  "next -> main" PR opens automatically
```

The `next→main` PR is a living PR: if more feature branches merge to `next` before the PR is reviewed, it accumulates all changes. There is always at most one open `next→main` PR per repository.

**The `next→main` PR must be reviewed and merged manually.** It is never merged automatically.

---

## What the Automations Do

| Trigger | What happens |
|---------|-------------|
| You create a `feature/*`, `fix/*`, `hotfix/*`, `chore/*`, `bugfix/*`, or `bug/*` branch from `main` | Branch is deleted automatically; you receive a comment with instructions |
| You open a PR to `main` from any branch other than `next` | PR is closed automatically; you receive a comment with instructions |
| Any PR is opened on any repo | PR is added to the [Nexus Issue Tracker](https://github.com/orgs/Nexus-PLM/projects/1) project board |
| A commit lands on `next` (e.g., PR merge) | A `next→main` PR is opened automatically if one is not already open |

---

## CI Behaviour

All repositories with a `build.yml` run CI on:
- Every push to `main`
- Every push to `next`
- Every pull request targeting `main` or `next`

A PR cannot be merged if CI is failing.

---

## Branch Naming Reference

| Branch type | Format | Example |
|-------------|--------|---------|
| New feature | `feature/<issue-n>-<slug>` | `feature/8-repo-refactoring` |
| Bug fix | `bugfix/<issue-n>-<slug>` or `bug/<issue-n>-<slug>` | `bugfix/14-null-ref-on-load` |
| Small fix | `fix/<issue-n>-<slug>` | `fix/22-typo-in-error-message` |
| Hotfix | `hotfix/<issue-n>-<slug>` | `hotfix/31-crash-on-startup` |
| Maintenance | `chore/<issue-n>-<slug>` | `chore/9-update-dependencies` |
| Integration | `next` | — |
| Production | `main` | — |

All of the above working branch prefixes (`feature/`, `fix/`, `hotfix/`, `chore/`, `bugfix/`, `bug/`) are enforced — any such branch created from `main` instead of `next` is automatically deleted.

---

## Project Board

All open issues and pull requests across every Nexus-PLM repository are tracked in one place:

**[Nexus Issue Tracker → https://github.com/orgs/Nexus-PLM/projects/1](https://github.com/orgs/Nexus-PLM/projects/1)**

PRs are added automatically. Issues should be added manually when work is planned.

---

## Emergency Access (Org Admins Only)

If a direct push to `main` or `next` is ever required (e.g., a critical infrastructure fix), an org admin can temporarily disable enforcement:

```bash
# Disable enforce_admins for one branch on one repo
gh api repos/Nexus-PLM/<repo>/branches/<branch>/protection/enforce_admins -X DELETE

# ... make the necessary change ...

# Re-enable immediately after
gh api repos/Nexus-PLM/<repo>/branches/<branch>/protection/enforce_admins -X POST -f enabled=true
```

Re-enable protection as soon as the change is made. Do not leave it disabled.

**Important — keep `next` in sync after any direct commit to `main`:**

If you commit directly to `main` (e.g., updating a workflow file), immediately merge `main` into `next` to prevent branch divergence:

```bash
# Temporarily bypass next protection
gh api repos/Nexus-PLM/<repo>/branches/next/protection/enforce_admins -X DELETE

# Merge main into next
gh api repos/Nexus-PLM/<repo>/merges -X POST \
  -f base=next -f head=main \
  -f commit_message="chore: sync main into next"

# Re-enable next protection
gh api repos/Nexus-PLM/<repo>/branches/next/protection/enforce_admins -X POST -f enabled=true
```

Never commit the same content independently to both `main` and `next` — this creates diverged histories that are difficult to reconcile. Always commit to one branch and propagate via merge.

---

## Quick Reference

```
┌─ Have an issue? ──────────────────────────────────────────────┐
│                                                                │
│  1. Find/create issue in the project board                     │
│  2. .\05-create-branch-from-issue.ps1 -IssueUrl <url>         │
│  3. git fetch && git checkout feature/<n>-<slug>              │
│  4. Make changes, push                                         │
│  5. Open PR → next                                             │
│  6. Merge PR (CI must pass)                                    │
│  7. next→main PR opens automatically — review and merge when   │
│     ready to release                                           │
└───────────────────────────────────────────────────────────────┘

DO:     Branch from next
DO:     PR to next
DO:     Let the automation open next→main PRs

DON'T:  Branch from main        → branch gets deleted (feature/, fix/, hotfix/, chore/, bugfix/, bug/)
DON'T:  PR directly to main     → PR gets closed
DON'T:  Push directly to next   → push is rejected
DON'T:  Push directly to main   → push is rejected
```
