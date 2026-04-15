---
inclusion: always
---

# EC Europa Validation in Worktrees

## Overview

All quality checks run inside the Docker `web` container. Because `.auto-pilot/` is inside the Docker volume mount, worktree paths are accessible from the container without any extra configuration.

## PHPCS — Coding Standards

```bash
# Check a module in the worktree
docker compose exec web vendor/bin/run toolkit:test-phpcs --paths=.auto-pilot/task-001/lib/modules/

# Direct invocation (fallback)
docker compose exec web vendor/bin/phpcs --standard=phpcs.xml .auto-pilot/task-001/lib/modules/

# Auto-fix violations
docker compose exec web vendor/bin/phpcbf --standard=phpcs.xml .auto-pilot/task-001/lib/modules/
```

## PHPStan — Static Analysis

```bash
# Analyse a module in the worktree
docker compose exec web vendor/bin/phpstan analyse .auto-pilot/task-001/lib/modules/my_module/

# With project config
docker compose exec web vendor/bin/phpstan analyse --configuration=phpstan.neon .auto-pilot/task-001/lib/modules/
```

## PHPMD — Mess Detection

```bash
docker compose exec web vendor/bin/phpmd .auto-pilot/task-001/lib/modules/ text phpmd.xml
```

## PHPUnit — Tests

```bash
# Run tests for a specific module in the worktree
docker compose exec web vendor/bin/phpunit .auto-pilot/task-001/lib/modules/my_module/tests/

# Run by group
docker compose exec web vendor/bin/phpunit --group=my_module .auto-pilot/task-001/lib/modules/

# With coverage
docker compose exec web vendor/bin/phpunit --coverage-text .auto-pilot/task-001/lib/modules/my_module/tests/
```

## AI Validation Commands (project-specific)

If your project has custom AI validation commands (e.g., a Drush command or custom script), run them after the toolkit checks:

```bash
# Example — replace with your project's validation command
docker compose exec web drush ai:validate-code phpcs $MODULE_PATH
```

If no custom validation exists, the toolkit checks above are sufficient.

## Full Pre-MR Validation Sequence

Run all checks before pushing and creating the MR:

```bash
WORKTREE=".auto-pilot/task-001"
MODULE_PATH="$WORKTREE/lib/modules/my_module"

# 1. Coding standards
docker compose exec web vendor/bin/run toolkit:test-phpcs --paths=$MODULE_PATH \
  || { echo "PHPCS failed"; exit 1; }

# 2. Static analysis
docker compose exec web vendor/bin/phpstan analyse $MODULE_PATH \
  || { echo "PHPStan failed"; exit 1; }

# 3. Mess detection
docker compose exec web vendor/bin/phpmd $MODULE_PATH text phpmd.xml \
  || { echo "PHPMD failed"; exit 1; }

# 4. Unit tests
docker compose exec web vendor/bin/phpunit $MODULE_PATH/tests/ \
  || { echo "PHPUnit failed"; exit 1; }

echo "All checks passed."
```

## Quality Thresholds

| Check | Threshold |
|-------|-----------|
| PHPCS violations | Zero |
| PHPStan errors | Zero (at configured level) |
| PHPMD violations | Zero |
| Unit test coverage (new code) | ≥ 90% |
| All tests | Passing |
