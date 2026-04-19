# Contributing to Nexus PLM -- Branching & Workflow Guide

This document describes how all code changes must flow through the Nexus-PLM repositories. Read it before opening your first pull request.

---

## The Three-Tier Model

```
feature/<n>-<slug>  --PR--►  next  --auto-PR--►  main
```

| Branch | Role | Protected? |
|--------|------|-----------|
| `main` | Production -- what is deployed/released | ✅ Yes |
| `next` | Integration / UAT -- what is being prepared for release | ✅ Yes |
| `feature/*` | Your working branch | [!] No |

**Rules:**
- `feature/*` branches are created from `next`, not from `main`
- Changes reach `next` via a pull request from a `feature/*` branch
- Changes reach `main` via a pull request from `next` only -- never directly
- Direct push to `main` or `next` is blocked for everyone, including org admins

---

## Step-by-Step: Contributing a Change

### 1. Find or create an issue

All work should be tracked on the [Nexus Issue Tracker](https://github.com/orgs/Nexus-PLM/projects/1). Open an issue in the relevant repository before starting work, or pick up an existing open issue.

### 2. Create your feature branch

Use the branch creation script -- it names your branch correctly and points it at `next`:

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

Work on your `feature/*` branch as normal. Commit as often as you like -- only the PR merge matters for the shared branches.

```bash
git add .
git commit -m "your message"
git push origin feature/<n>-<slug>
```

### 4. Open a pull request to `next`

Open a PR from your `feature/*` branch targeting **`next`** (not `main`).

When the PR is opened:
- The title is automatically prefixed: `PR to next: <your title>`
- It is automatically added to the [Nexus Issue Tracker](https://github.com/orgs/Nexus-PLM/projects/1) project board
- CI runs against the `next` branch trigger

When the PR is merged, its project board item status is automatically set to **Done**.
If the PR is closed without merging, the status is set to **Cancelled**.

If your PR description contains `Closes #<n>` or `Fixes #<n>`, the linked issue will be automatically closed when the PR merges.

> **Do not** open a PR directly to `main`. If you do, it will be automatically closed with an explanation comment.

### 5. Get your PR reviewed and merged

Once your `feature/*` PR is approved and merged into `next`, your changes are in the integration branch.

### 6. Promotion to main (automatic)

Merging anything into `next` automatically triggers a pull request from `next` to `main`. You do not open this PR -- it appears on its own within seconds, titled `PR to main: next -> main`.

```
Your PR merges to next  ->  "PR to main: next -> main" opens automatically
```

The `next->main` PR is a living PR: if more feature branches merge to `next` before the PR is reviewed, it accumulates all changes. There is always at most one open `next->main` PR per repository.

**The `next->main` PR must be reviewed and merged manually.** It is never merged automatically.

When the `next->main` PR is merged, its project board item is automatically set to **Done**.

---

## What the Automations Do

| Trigger | What happens |
|---------|-------------|
| You create a `feature/*`, `fix/*`, `hotfix/*`, `chore/*`, `bugfix/*`, or `bug/*` branch from `main` | Branch is deleted automatically; you receive a comment with instructions |
| You open a PR to `main` from any branch other than `next` | PR is closed automatically; you receive a comment with instructions |
| Any PR is opened on any repo | Title prefixed `PR to <base>: <title>`; PR added to project board in the **Pull Requests to `next`** or **Pull Requests to `main`** column; PR assigned to its author |
| A commit lands on `next` (e.g., PR merge) | A `PR to main: next -> main` PR is opened automatically, assigned to the pusher, and placed in the **Pull Requests to `main`** column |
| A PR is merged | Project board item status is set to **Done** |
| A PR is closed without merging | Project board item status is set to **Cancelled** |
| Any workflow run fails | A GitHub Issue is created and placed in the **Failed Actions** column of the project board |

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
| Integration | `next` | -- |
| Production | `main` | -- |

All of the above working branch prefixes (`feature/`, `fix/`, `hotfix/`, `chore/`, `bugfix/`, `bug/`) are enforced -- any such branch created from `main` instead of `next` is automatically deleted.

---

## Project Board

All open issues and pull requests across every Nexus-PLM repository are tracked in one place:

**[Nexus Issue Tracker -> https://github.com/orgs/Nexus-PLM/projects/1](https://github.com/orgs/Nexus-PLM/projects/1)**

### Columns

| Column | What goes there |
|--------|----------------|
| **Backlog** | Open issues waiting to be worked on |
| **Pull Requests to `next`** | PRs from `feature/*` branches targeting `next` |
| **Pull Requests to `main`** | Auto-generated `next -> main` promotion PRs |
| **Failed Actions** | GitHub Issues auto-created when a CI workflow fails |
| **Done** | Merged PRs and resolved issues |
| **Cancelled** | Closed-without-merging PRs |

### PR lifecycle in the board

| Event | Title in board | Column |
|-------|---------------|--------|
| PR opened (feature -> next) | `PR to next: <your title>` | Pull Requests to `next` |
| PR opened (next -> main, auto) | `PR to main: next -> main` | Pull Requests to `main` |
| PR merged | -- | **Done** |
| PR closed without merging | -- | **Cancelled** |
| Build/Test workflow fails | `Action failed: <workflow> on <branch> (<sha>)` | **Failed Actions** |

All status changes are automatic -- you do not need to update the board manually.

Issues should be added to the board manually when work is planned. PRs and failure alerts are added automatically.

---

## Creating a New Repository

When a new repository is created in the Nexus-PLM org, several automations must be set up before it is ready for the branching workflow.

### What needs to be configured

| Setup item | Why |
|-----------|-----|
| `next` branch created from `main` HEAD | Required for the three-tier workflow |
| Branch protection on `main` and `next` | Blocks direct pushes; requires PRs |
| `auto-pr-next-to-main.yml` on `next` | Opens `next -> main` PR automatically on every push |
| `add-pr-to-project.yml` on `main` | Adds PRs to the project board, prefixes titles, assigns authors |
| `mark-project-item-done.yml` on `main` | Sets PR board item to Done/Cancelled on close |
| `report-workflow-failure.yml` on `main` | Creates a Failed Actions issue when CI fails |
| `enforce-feature-base.yml` on `main` | Deletes branches created from `main` instead of `next` |
| `enforce-pr-to-main.yml` on `main` | Closes PRs targeting `main` from non-`next` branches |
| `NEXUS_PLM_PROJECT_TOKEN` secret | Used by board/PR automation workflows |
| `NEXUS_PLM_NUGET_TOKEN` secret | Used by `build.yml` to restore private NuGet packages (CI repos only) |

### Automated setup script

Run `07-setup-new-repo.ps1` to bootstrap a new repository in one shot:

```powershell
cd C:\GitHub\Nexus.PLM.Scripts\Branching-Strategy
.\07-setup-new-repo.ps1 -RepoName Nexus.PLM.MyNewRepo
```

This script will:
1. Create the `next` branch from `main` HEAD
2. Deploy all six automation workflow files
3. Set `NEXUS_PLM_PROJECT_TOKEN` secret
4. Apply branch protection to `main` and `next`

If the repo has a `build.yml` CI workflow, also run:
```powershell
.\07-setup-new-repo.ps1 -RepoName Nexus.PLM.MyNewRepo -HasCI
```

The `-HasCI` flag additionally sets `NEXUS_PLM_NUGET_TOKEN` on the repo.

### Manual checklist (if not using the script)

```
☐ gh api repos/Nexus-PLM/<repo>/git/refs -X POST -f ref=refs/heads/next -f sha=<main-sha>
☐ Deploy add-pr-to-project.yml, mark-project-item-done.yml, report-workflow-failure.yml,
     enforce-feature-base.yml, enforce-pr-to-main.yml  ->  main, then sync next
☐ Deploy auto-pr-next-to-main.yml  ->  next only
☐ gh secret set NEXUS_PLM_PROJECT_TOKEN --repo Nexus-PLM/<repo>
☐ Apply branch protection (PUT /branches/main/protection and /branches/next/protection)
☐ (if CI repo) gh secret set NEXUS_PLM_NUGET_TOKEN --repo Nexus-PLM/<repo>
```

---

## AI Agents (Admin Tools)

The `C:\GitHub\Nexus.PLM.Scripts\Agents\` directory contains Claude AI agent definitions for automating administrative tasks. Each agent lives in its own subdirectory with instructions, scripts, and a README.

### Available Agents

| Agent | Directory | Purpose |
|-------|-----------|---------|
| Investigate Failed Actions | `Agents\investigate-failed-actions\` | Queries the **Failed Actions** project board column, pulls GitHub Actions logs, diagnoses root causes, and fixes infrastructure issues or documents developer fixes needed |

### Using an Agent

Open Claude Code in the Scripts repo directory and ask it to:

```
Read C:\GitHub\Nexus.PLM.Scripts\Agents\investigate-failed-actions\AGENT.md and follow the procedure.
```

Or use the standalone scripts in the agent directory directly:

```powershell
cd C:\GitHub\Nexus.PLM.Scripts\Agents\investigate-failed-actions

# List all Failed Actions items from the project board
.\get-failed-actions.ps1

# Close a resolved issue and move to Done on the board
.\close-resolved.ps1 -Repo Nexus.PLM.Core -IssueNumber 5 -ItemId PVTI_xxx
```

### Agent Token Requirements

Agents that interact with GitHub need `GH_TOKEN` set with scopes: `repo`, `workflow`, `project`.

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

**Important -- keep `next` in sync after any direct commit to `main`:**

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

Never commit the same content independently to both `main` and `next` -- this creates diverged histories that are difficult to reconcile. Always commit to one branch and propagate via merge.

---

## Quick Reference

```
+- Have an issue? ----------------------------------------------+
|                                                                |
|  1. Find/create issue in the project board                     |
|  2. .\05-create-branch-from-issue.ps1 -IssueUrl <url>         |
|  3. git fetch && git checkout feature/<n>-<slug>              |
|  4. Make changes, push                                         |
|  5. Open PR -> next                                             |
|  6. Merge PR (CI must pass)                                    |
|  7. next->main PR opens automatically -- review and merge when   |
|     ready to release                                           |
+---------------------------------------------------------------+

DO:     Branch from next
DO:     PR to next
DO:     Let the automation open next->main PRs

DON'T:  Branch from main        -> branch gets deleted (feature/, fix/, hotfix/, chore/, bugfix/, bug/)
DON'T:  PR directly to main     -> PR gets closed
DON'T:  Push directly to next   -> push is rejected
DON'T:  Push directly to main   -> push is rejected
```
