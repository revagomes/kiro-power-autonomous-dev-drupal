# kiro-power-autonomous-dev-drupal

A [Kiro Power](https://kiro.dev/blog/introducing-powers/) — EC Europa / Drupal profile for autonomous parallel development.

This power layers EC Europa-specific conventions on top of [kiro-power-autonomous-dev](https://github.com/revagomes/kiro-power-autonomous-dev). **Install both powers** for the full experience.

## What it adds

- **JIRA commit format** — enforced by the EC Europa Toolkit `commit-msg` hook
- **GitLab MR creation** — via `glab` CLI or GitLab REST API
- **Docker-scoped validation** — PHPCS, PHPStan, PHPMD, PHPUnit inside the `web` container
- **EC Europa Toolkit quality gates** — `vendor/bin/run toolkit:*` commands
- **Drupal-aware adversarial review gate** — a skeptical reviewer that checks for `\Drupal::` static calls, unexported config, module-prefix/location, and the toolkit quality thresholds before any MR is opened
- **`ai-ready` label workflow** — autonomous GitLab issue pickup with claim/release/timeout

## Slash commands

Overrides two commands from the generic power:

| Command | Override |
|---------|---------|
| `/auto-pilot run` | Uses EC Europa Toolkit validation + Drupal adversarial review gate + JIRA commit format |
| `/auto-pilot fetch` | Fetches GitLab issues labeled `ai-ready` |

`dry-run`, `status`, and `clean` are inherited from the generic power unchanged.

## Install

Install both powers in Kiro IDE:

1. `https://github.com/revagomes/kiro-power-autonomous-dev`
2. *(this repo)*

## Structure

```
POWER.md
prompts/
  auto-pilot-run.prompt.md      ← overrides generic run with EC Europa validation + review gate
  auto-pilot-fetch.prompt.md    ← overrides generic fetch with GitLab ai-ready workflow
steering/
  jira-commit-format.md
  ec-europa-validation.md
  gitlab-mr-workflow.md
  ai-ready-issue-workflow.md
agents/
  ec-europa-adversarial-reviewer.md   ← Drupal-aware pre-MR review gate
```

## Requirements

- EC Europa Toolkit project (`vendor/bin/run` available)
- Docker Compose environment (`fpfis/httpd-php` web container)
- GitLab instance with API access
- `GITLAB_TOKEN`, `GITLAB_URL`, `GITLAB_PROJECT_ID` in `.worktrees/.env`

## License

MIT
