---
inclusion: always
---

# GitLab MR Workflow

## Creating a Merge Request

After pushing the branch, create an MR using `glab` CLI (preferred) or the REST API.

### Using `glab` CLI

```bash
source .auto-pilot/.env

glab mr create \
  --source-branch feature/task-001-registration-validation \
  --target-branch $AUTOPILOT_BASE_BRANCH \
  --title "PROJ-123: Add input validation to registration form." \
  --description "## Summary
Add email format and password strength validation to the user registration form.

## Changes
- \`lib/modules/my_module/src/Service/RegistrationValidator.php\` — New validation service
- \`lib/modules/my_module/tests/src/Unit/Service/RegistrationValidatorTest.php\` — 12 unit tests

## Testing
All unit tests pass:
\`\`\`bash
docker compose exec web vendor/bin/phpunit lib/modules/my_module/tests/
\`\`\`" \
  --remove-source-branch \
  --assignee "@me"
```

### Using GitLab REST API

```bash
source .auto-pilot/.env

curl --silent --fail \
  --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  --header "Content-Type: application/json" \
  --data "{
    \"source_branch\": \"feature/task-001-registration-validation\",
    \"target_branch\": \"$AUTOPILOT_BASE_BRANCH\",
    \"title\": \"PROJ-123: Add input validation to registration form.\",
    \"description\": \"## Summary\nAdd email format and password strength validation.\n\n## Changes\n- lib/modules/my_module/src/Service/RegistrationValidator.php\n\n## Testing\nAll unit tests pass.\",
    \"remove_source_branch\": true,
    \"labels\": \"ai-review-ready\"
  }" \
  "$GITLAB_URL/api/v4/projects/$GITLAB_PROJECT_ID/merge_requests"
```

## MR Title Format

MR titles must follow the same JIRA commit format:

```
<JIRA-ID>: <Capital letter description ending with a period.>
```

Example: `PROJ-123: Add input validation to registration form.`

## MR Description Template

```markdown
## Summary
<One paragraph describing what was implemented and why.>

## Changes
- `lib/modules/my_module/src/...` — <what changed>
- `lib/modules/my_module/tests/...` — <what was tested>

## Testing
<Exact commands to verify the changes:>
```bash
docker compose exec web vendor/bin/phpunit lib/modules/my_module/tests/
```

## Notes
<Decisions made, trade-offs, or follow-up work needed.>
```

## Labels

| Label | Set by | Meaning |
|-------|--------|---------|
| `ai-ready` | Human | Issue is eligible for autonomous pickup |
| `ai-in-progress` | Agent | Agent has claimed the issue |
| `ai-review-ready` | Agent | MR created, ready for human review |

When creating the MR, set `labels: "ai-review-ready"` and remove `ai-in-progress` from the issue.

## Required Environment Variables

```bash
# .auto-pilot/.env
GITLAB_TOKEN=<your-private-access-token>
GITLAB_URL=https://gitlab.example.com
GITLAB_PROJECT_ID=<numeric-project-id>
```

Never commit these values. The `.auto-pilot/` directory is gitignored.
