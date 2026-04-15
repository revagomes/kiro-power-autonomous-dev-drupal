# kiro-power-autonomous-dev-drupal

A [Kiro Power](https://kiro.dev/blog/introducing-powers/) — EC Europa / Drupal profile for autonomous parallel development.

This power layers EC Europa-specific conventions on top of [kiro-power-autonomous-dev](https://github.com/revagomes/kiro-power-autonomous-dev). **Install both powers** for the full experience.

## What it adds

On top of the generic autonomous development workflow:

- **JIRA commit format** — enforced by the EC Europa Toolkit `commit-msg` hook
- **GitLab MR creation** — via `glab` CLI or GitLab REST API
- **Docker-scoped validation** — PHPCS, PHPStan, PHPMD, PHPUnit inside the `web` container
- **EC Europa Toolkit quality gates** — `vendor/bin/run toolkit:*` commands
- **`ai-ready` label workflow** — autonomous GitLab issue pickup with claim/release/timeout

## Install

Install both powers in Kiro IDE:

1. `https://github.com/revagomes/kiro-power-autonomous-dev`
2. `https://github.com/revagomes/kiro-power-autonomous-dev-drupal`

## Steering files

| File | Purpose |
|------|---------|
| `jira-commit-format.md` | JIRA commit format, cadence, hook rules |
| `ec-europa-validation.md` | Docker-scoped PHPCS, PHPStan, PHPMD, PHPUnit |
| `gitlab-mr-workflow.md` | GitLab MR creation via `glab` or REST API |
| `ai-ready-issue-workflow.md` | Issue pickup, claim, release, timeout, failure logging |

## Requirements

- EC Europa Toolkit project (`vendor/bin/run` available)
- Docker Compose environment (`fpfis/httpd-php` web container)
- GitLab instance with API access
- `GITLAB_TOKEN`, `GITLAB_URL`, `GITLAB_PROJECT_ID` in `.auto-pilot/.env`

## Related

- [kiro-power-autonomous-dev](https://github.com/revagomes/kiro-power-autonomous-dev) — generic orchestration engine (required)

## License

MIT
