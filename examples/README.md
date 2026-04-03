# Examples

## Framework-Specific CLAUDE.md Templates

These are complete, copy-ready CLAUDE.md files with the behavioral rules already filled in plus framework-specific stack rules. Copy one to your project root and customize.

| Template | Stack | Copy Command |
|----------|-------|-------------|
| [Python](claude-md-python.md) | Python, FastAPI, Django, SQLAlchemy, Alembic | `cp examples/claude-md-python.md your-project/CLAUDE.md` |
| [React](claude-md-react.md) | React, Next.js, TypeScript, Tailwind, Zod | `cp examples/claude-md-react.md your-project/CLAUDE.md` |
| [Go](claude-md-go.md) | Go, Gin/Echo, GORM/sqlx, golangci-lint | `cp examples/claude-md-go.md your-project/CLAUDE.md` |
| [Rails](claude-md-rails.md) | Ruby on Rails, RSpec, Rubocop, Sidekiq | `cp examples/claude-md-rails.md your-project/CLAUDE.md` |

## Settings Template

| Template | Purpose |
|----------|---------|
| [settings-template.json](settings-template.json) | Full `~/.claude/settings.json` with all hooks, permissions, and env vars wired |

## How to Use

1. Pick the template closest to your stack
2. Copy it to your project root as `CLAUDE.md`
3. Customize the Stack Rules section -- add your project's conventions, remove rules that don't apply
4. The behavioral rules (Verify-After-Complete, Diagnose-First, Plan-Before-Execute) are universal -- keep them as-is

Don't see your framework? Check [PRESETS.md](../PRESETS.md#stack-rule-templates) for more stack rule snippets, or submit a PR with your own template.
