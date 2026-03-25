# Benchmarks

Real-world performance data from production use of this blueprint. Results vary by workflow, project size, and usage patterns.

> **Disclaimer:** These numbers come from one developer's workflow across multiple projects. Your results will differ based on your stack, session length, and usage patterns. Use these as directional guidance, not guarantees.

---

## Token Efficiency

| Metric | Without Blueprint | With Blueprint | Improvement |
|--------|------------------|----------------|-------------|
| Playwright browser automation | ~114K tokens/task (MCP) | ~27K tokens/task (CLI) | **-76%** |
| Context preservation across compaction | Lost (no hooks) | Serialized to disk (PreCompact hook) | Prevents re-reading files |
| Redundant permission prompts | ~10-20 per session | 0 (allow list + auto mode) | Eliminates prompt fatigue |
| Agent context isolation | Shared (polluted) | Worktree isolation (fresh context) | Cleaner reviews |

### Playwright MCP vs CLI Detail

The Playwright MCP server streams full DOM accessibility trees into the context window on every interaction. After 15 browser actions, you're carrying 60-80K tokens of accumulated DOM state.

Running `npx playwright test` via the Bash tool saves results to disk. The model reads only the summary/failures, not the full DOM. This architectural difference produces the 76% token reduction.

**Recommendation:** Use MCP for interactive browser exploration. Use CLI (`npx playwright test`) for test execution.

---

## Cost

Based on [Anthropic's pricing](https://docs.anthropic.com/en/docs/about-claude/pricing) as of March 2026.

| Usage Pattern | Estimated Daily Cost | Notes |
|--------------|---------------------|-------|
| Light (5-10 responses, no agents) | ~$1-3 | CLAUDE.md + hooks only |
| Standard (20-50 responses, occasional agents) | ~$5-8 | Standard preset |
| Heavy (100+ responses, parallel agents) | ~$10-15 | Full preset with review skill |
| Stop hook overhead | ~$1-2/day additional | Sonnet security review on every response |

### Model Tiering Impact

| Configuration | Relative Cost | Description |
|--------------|---------------|-------------|
| All Opus agents | 5x baseline | Every agent uses the most expensive model |
| All Sonnet agents | 1x baseline | Standard pricing |
| Blueprint tiering (1 Opus + 8 Sonnet + 2 Haiku) | ~1.1x baseline | Minimal premium for architecture quality |

The blueprint's model tiering keeps costs close to all-Sonnet pricing while reserving Opus reasoning for architecture decisions (project-architect agent only).

---

## Quality

| Metric | Without Blueprint | With Blueprint | Enforcement |
|--------|------------------|----------------|-------------|
| Config accidentally weakened | Risk on every edit | **Blocked** | `protect-config.sh` (PreToolUse hook) |
| Accidental push to wrong remote | Risk on every push | **Blocked** | `block-git-push.sh` (PreToolUse hook) |
| Post-edit verification forgotten | Common | **Automatic reminder** | `notify-file-changed.sh` (PostToolUse hook) |
| Post-commit review skipped | Common | **Automatic reminder** | `post-commit-review.sh` (PostToolUse hook) |
| Plan verification thoroughness | Self-review only (~80%) | **7-point mechanical check** (100%) | `verify-plan` agent |
| Security review on every response | None | **Automatic** (SQL injection, XSS, secrets) | Stop hook (Sonnet model) |
| Session context lost on crash | Lost completely | **Serialized to disk** | PreCompact + SessionEnd hooks |
| Cost tracking | Unknown spending | **JSONL metrics per session** | `cost-tracker.sh` |

### Hook Compliance: 100% vs ~80%

CLAUDE.md instructions are followed approximately 80% of the time (the model occasionally forgets or deprioritizes rules). Hooks execute deterministically on every matching event -- 100% compliance, zero exceptions. This is why enforcement belongs in hooks, and guidance belongs in CLAUDE.md.

---

## How to Measure Your Own

1. **Token usage:** Check your Anthropic dashboard or use the `cost-tracker.sh` hook data at `~/.claude/metrics/costs.jsonl`
2. **Hook blocks:** Add logging to `block-git-push.sh` and `protect-config.sh` to count how often they prevent mistakes
3. **Session cost:** Compare daily Anthropic bills before and after adopting the blueprint (give it a week for meaningful data)
4. **Quality:** Track how many bugs are caught by the Stop hook's security review vs found later in production

---

## Contributing Your Data

If you've measured your own before/after numbers, we'd love to include them. Open a [Discussion](../../discussions) or submit a PR with your data point. Anonymous data is fine -- we care about the numbers, not the project name.
