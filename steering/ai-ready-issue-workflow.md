---
inclusion: always
---

# AI-Ready Issue Workflow

## Overview

Issues labeled `ai-ready` in GitLab are eligible for autonomous pickup. The orchestrator polls for these issues, claims them, implements a solution, and creates an MR for human review.

> **Status**: The GitLab polling loop is planned. The manual task YAML workflow (from `kiro-power-autonomous-dev`) is the current implementation. This file documents the full intended workflow for when polling is enabled.

> The runtime directory is `.worktrees/` (renamed from `.auto-pilot/` so `.env` is never tracked). See `POWER.md`.

## Issue Eligibility

An issue is eligible for autonomous pickup if:
- It has the `ai-ready` label
- It has no assignee
- It does not have `ai-in-progress` or `ai-review-ready` labels
- Its complexity is within the allowed threshold (see below)

## Complexity Ceiling

Agents must not pick up architectural, security, or high-risk tasks. Before claiming an issue, evaluate:

**Eligible (low complexity):**
- Adding a new field to an existing content type
- Writing missing unit tests for an existing service
- Fixing a PHPCS or PHPStan violation
- Adding a new translation string
- Updating a configuration value

**Not eligible (high complexity or risk):**
- Changing authentication or authorization logic
- Modifying database schema in a breaking way
- Refactoring a core service used by many modules
- Any task touching security-sensitive configuration (roles, permissions, authentication settings)
- Tasks with vague acceptance criteria ("improve performance", "refactor X")

If unsure, do not claim the issue. Leave it for a human.

## Claim Workflow

```bash
source .worktrees/.env

ISSUE_ID=123
PROJECT_ID=$GITLAB_PROJECT_ID

# 1. Assign the issue to the agent (claim it)
curl --silent --show-error --fail-with-body \
  --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  --request PUT \
  "$GITLAB_URL/api/v4/projects/$PROJECT_ID/issues/$ISSUE_ID" \
  --data "assignee_ids[]=<agent-user-id>"

# 2. Add ai-in-progress label, remove ai-ready
curl --silent --show-error --fail-with-body \
  --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  --request PUT \
  "$GITLAB_URL/api/v4/projects/$PROJECT_ID/issues/$ISSUE_ID" \
  --data "add_labels=ai-in-progress&remove_labels=ai-ready"
```

## Fetch Eligible Issues

```bash
# Fetch all ai-ready issues with no assignee
curl --silent --show-error --fail-with-body \
  --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  "$GITLAB_URL/api/v4/projects/$GITLAB_PROJECT_ID/issues?labels=ai-ready&assignee_id=None&state=opened" \
  | jq '.[] | {id: .iid, title: .title, description: .description}'
```

The `/auto-pilot fetch` command writes eligible issues to `.worktrees/tasks.yaml` in the standard format.

## After MR Creation

```bash
# Remove ai-in-progress, add ai-review-ready
curl --silent --show-error --fail-with-body \
  --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  --request PUT \
  "$GITLAB_URL/api/v4/projects/$PROJECT_ID/issues/$ISSUE_ID" \
  --data "add_labels=ai-review-ready&remove_labels=ai-in-progress"
```

## Timeout and Release

If no MR is created within the configured timeout (default: 2 hours):

```bash
# Release the issue — remove ai-in-progress, restore ai-ready, unassign
curl --silent --show-error --fail-with-body \
  --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  --request PUT \
  "$GITLAB_URL/api/v4/projects/$PROJECT_ID/issues/$ISSUE_ID" \
  --data "assignee_ids[]=0&add_labels=ai-ready&remove_labels=ai-in-progress"
```

Log the timeout to `.worktrees/orchestrator.log` with the issue ID and reason.

## Human Override

A human can reclaim any issue at any time by:
- Removing the `ai-ready` label
- Assigning themselves to the issue

The agent must check for this before each commit and stop if the issue is no longer assigned to it.

## Failed Attempts

If implementation fails, log a comment on the issue before releasing it:

```bash
curl --silent --show-error --fail-with-body \
  --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  --header "Content-Type: application/json" \
  --data "{\"body\": \"Autonomous implementation attempt failed.\n\nReason: PHPStan errors could not be resolved — type mismatch in RegistrationValidator::validate().\n\nThe issue has been released for human pickup.\"}" \
  "$GITLAB_URL/api/v4/projects/$PROJECT_ID/issues/$ISSUE_ID/notes"
```

Never silently abandon a failed attempt. When logging the failure comment, keep it concise and free of internal infrastructure details — the comment is public-facing.
