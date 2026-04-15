---
name: "autonomous-dev-drupal"
displayName: "Autonomous Development — EC Europa / Drupal Profile"
description: "EC Europa Toolkit profile for autonomous parallel development: JIRA commit format, GitLab MR workflow, Docker-scoped validation, and ai-ready issue pickup"
keywords: ["autonomous", "drupal", "ec europa", "gitlab", "jira", "worktree", "parallel", "orchestrator", "phpcs", "phpstan", "toolkit", "merge request", "ai-ready"]
author: "Reva Gomes"
---

# Autonomous Development — EC Europa / Drupal Profile

## Overview

This power is a profile on top of [kiro-power-autonomous-dev](https://github.com/revagomes/kiro-power-autonomous-dev). It adds EC Europa-specific conventions on top of the generic autonomous development workflow:

- **JIRA commit format** enforced by the EC Europa Toolkit `commit-msg` hook
- **GitLab MR creation** via `glab` CLI or GitLab REST API
- **Docker-scoped validation** — all quality checks run inside the `web` container
- **EC Europa Toolkit quality gates** — PHPCS, PHPStan, PHPMD via `vendor/bin/run`
- **`ai-ready` label workflow** for autonomous GitLab issue pickup

**Install both powers** for the full experience:
1. `kiro-power-autonomous-dev` — generic orchestration engine
2. `kiro-power-autonomous-dev-drupal` — this EC Europa profile

## Onboarding

### Step 1: Install the generic power first

Import `kiro-power-autonomous-dev` before this one. This profile adds EC Europa specifics on top — it does not replace the generic workflow.

### Step 2: Configure GitLab credentials

Add to `.auto-pilot/.env`:

```bash
AUTOPILOT_REMOTE=origin
AUTOPILOT_BASE_BRANCH=main

# GitLab
GITLAB_TOKEN=<your-private-token>
GITLAB_URL=https://gitlab.example.com
GITLAB_PROJECT_ID=<numeric-project-id>
```

### Step 3: Understand the Docker command pattern

All validation commands run inside the Docker `web` container. The `.auto-pilot/` directory is inside the Docker volume mount, so no extra configuration is needed:

```bash
# Validate code in a worktree
docker compose exec web vendor/bin/run toolkit:test-phpcs --paths=.auto-pilot/task-001/lib/modules/

# Run PHPUnit for a module in the worktree
docker compose exec web vendor/bin/phpunit .auto-pilot/task-001/lib/modules/my_module/tests/
```

### Step 4: Label issues for autonomous pickup

In GitLab, add the `ai-ready` label to issues that are eligible for autonomous implementation. The orchestrator will pick them up, implement, and create an MR.

## When to Load Steering Files

- Commit message format and cadence → `jira-commit-format.md`
- Running EC Europa quality gates in worktrees → `ec-europa-validation.md`
- Creating GitLab MRs → `gitlab-mr-workflow.md`
- Picking up and processing `ai-ready` issues → `ai-ready-issue-workflow.md`

## EC Europa Conventions

### Commit format (enforced by hook)

```
<JIRA-ID>: <Capital letter subject ending with a period.>
```

Example: `PROJ-123: Add validation service for user registration.`

### Module prefix

Custom modules use a project-specific prefix (e.g., `my_`, `ejp_`). All custom code lives in `lib/modules/`.

### Config as code

Never modify Drupal configuration without exporting it. After any config change: `docker compose exec web drush cex`.

### No `\Drupal::` static calls

All services use dependency injection via the `create()` factory method.
