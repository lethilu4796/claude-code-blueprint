# Cross-Tool Guide — Using These Concepts Beyond Claude Code

While this blueprint is built for Claude Code, the **principles are universal**. This guide maps each concept to its equivalent in other AI coding tools, based on their official documentation as of early 2026.

## Quick Reference: Config File Locations

| Tool | Behavioral Rules | Main Config | User Config Directory | Custom Agents | MCP Config |
|------|-----------------|-------------|----------------------|---------------|------------|
| **Claude Code** | `CLAUDE.md` | `~/.claude/settings.json` | `~/.claude/` | `.claude/agents/*.md` | `.claude.json` or settings |
| **Cursor** | `.cursor/rules/*.mdc` | `.cursor/` directory | `~/.cursor/` | `.cursor/agents/*.md` | In Cursor settings |
| **Codex CLI** | `AGENTS.md` | `~/.codex/config.toml` | `~/.codex/` | Via Agents SDK | In `config.toml` |
| **Gemini CLI** | `GEMINI.md` | `~/.gemini/settings.json` | `~/.gemini/` | Subagent configs | In `settings.json` |
| **Windsurf** | `.windsurf/rules/*.md` + `AGENTS.md` | `~/.codeium/windsurf/` | `~/.codeium/windsurf/` | Not supported | `mcp_config.json` |

---

## Feature Comparison Matrix

| Feature | Claude Code | Cursor | Codex CLI | Gemini CLI | Windsurf |
|---------|:-----------:|:------:|:---------:|:----------:|:--------:|
| Behavioral rules file | Yes | Yes | Yes | Yes | Yes |
| Rules hierarchy (global + project) | Yes | Yes | Yes | Yes | Yes |
| Path-scoped rules | Yes | Yes (globs in `.mdc`) | Yes (directory walk) | Yes (directory walk) | Yes (glob triggers) |
| Custom subagents | Yes | Yes (v2.4+) | Yes | Yes (experimental) | No (parallel agents only) |
| Lifecycle hooks | Yes (full) | Yes (full) | Limited (prompt-level) | Yes (experimental) | Limited (enterprise audit) |
| MCP server support | Yes | Yes | Yes | Yes | Yes |
| Native memory persistence | Yes | Removed (use Rules) | Transcript resume | Yes (`save_memory`) | Yes (auto-generated) |
| Model tiering per-agent | Yes | No | Yes (per-config) | No | No |
| Permission system | Yes | No | Yes (sandbox modes) | No | No |
| Worktree isolation | Yes | No | No | No | Yes (parallel agents) |

---

## Translating the Blueprint

### 1. Behavioral Rules (Every Tool Has This)

This is the most portable concept. Every AI coding tool reads a project-level instruction file.

**Claude Code** → `CLAUDE.md` in project root

**Cursor** → `.cursor/rules/*.mdc` files with frontmatter
```markdown
---
description: Verify after completing any implementation
globs: ["**/*.ts", "**/*.tsx"]
alwaysApply: true
---
After finishing any implementation, always run a verification step...
```
Note: `.cursorrules` (root file) is deprecated but still works. The new `.mdc` format supports glob-scoped activation.

**Codex CLI** → `AGENTS.md` (hierarchical discovery)
Codex walks from `~/.codex/AGENTS.md` (global) down through project root to CWD, loading every `AGENTS.md` it finds. `AGENTS.override.md` takes precedence at each level.

**Gemini CLI** → `GEMINI.md` (hierarchical + imports)
Similar directory walk from `~/.gemini/GEMINI.md` (global) to project root (`.git` boundary). Supports `@file.md` imports for splitting large files into modules.

**Windsurf** → `.windsurf/rules/*.md` + `AGENTS.md`
Rules use frontmatter with `trigger` field (`glob`, `always_on`, `manual`). Root `AGENTS.md` files are always-on. Subdirectory `AGENTS.md` files auto-scope to that directory.

**What to copy:** Take the [CLAUDE.md](CLAUDE.md) template. Paste the rules into your tool's equivalent file. The content is tool-agnostic — only the file name and format change.

---

### 2. Subagents / Custom Agents (Claude Code + Cursor + Codex + Gemini)

Claude Code lets you create specialized subagents with their own instructions, model, and tool access.

**Cursor** (v2.4+) supports custom agents in `.cursor/agents/*.md` — similar concept with independent context and parallel execution.

**Codex CLI** supports subagents that spawn in parallel with independent context. Custom agents are configured via the OpenAI Agents SDK with model/instruction overrides.

**Gemini CLI** has experimental local and remote subagents with isolated context, independent history, and recursion protection.

**Windsurf** runs up to 5 parallel Cascade agents using Git worktrees (each gets its own branch), but does not support custom subagent definitions — they're full Cascade instances, not lightweight task-specific agents.

**What to copy:** The agent `.md` files from this blueprint can be adapted as custom agent definitions in Cursor (same format) or as instruction sets for Codex/Gemini subagents. The key pattern — separate agents for architecture, implementation, review, and testing — works in any tool.

---

### 3. Lifecycle Hooks (Claude Code + Cursor — Others Limited)

Hooks are deterministic automation that fires on specific events (before/after file edits, shell commands, session start/end, etc.).

**Claude Code** has the most complete hook system: SessionStart, PreToolUse, PostToolUse, PostToolUseFailure, PreCompact, PostCompact, Stop, SessionEnd — with matchers for specific tools.

**Cursor** (since Oct 2025) has hooks for: `beforeShellExecution`, `beforeMCPExecution`, `beforeReadFile`, `afterFileEdit`, `stop`. Configured via JSON, executed as standalone processes. Very similar to Claude Code's system.

**Codex CLI** has a `userpromptsubmit` hook that can block/augment prompts before execution. No file-edit or shell-execution lifecycle hooks.

**Gemini CLI** has experimental hooks for session context injection and active-agent tracking. Behind a toggle.

**Windsurf** has Cascade Hooks for logging and policy enforcement — enterprise/teams only, not general-purpose.

**What to copy:** The hook *scripts* in this blueprint (`protect-config.sh`, `block-git-push.sh`, etc.) can be repurposed as:
- **Git hooks** (`pre-push`, `post-commit`) that work with ANY tool
- **Cursor hooks** (direct port — same event model)
- **CI checks** for tools without hook support

---

### 4. Memory Persistence (Varies Significantly)

**Claude Code** has dual memory: auto-memory (`~/.claude/projects/*/memory/MEMORY.md`) + the git-backed system in this blueprint's [memory-template/](memory-template/).

**Cursor** briefly had a "Memories" feature but **removed it** starting v2.1.x. Users were told to export memories into Rules files. No built-in cross-session persistence. Community workarounds exist via MCP servers (Memory Banks, ContextForge).

**Codex CLI** saves session transcripts to `~/.codex/history.jsonl`. `codex resume` reopens earlier threads with same state. This is transcript replay, not semantic memory.

**Gemini CLI** has a `save_memory` tool that appends facts to `~/.gemini/GEMINI.md` under a `## Gemini Added Memories` section. Loaded automatically in subsequent sessions. Note: this writes directly into the same file used for instructions — a known limitation.

**Windsurf** auto-generates memories during conversations, stored in `~/.codeium/windsurf/memories/`. Workspace-scoped, persists across sessions. Users can also write durable facts to `.windsurf/rules/` files.

**What to copy:** The [memory-template/](memory-template/) pattern works with ANY tool:
1. Create a private git repo with the scaffold files
2. At session start, paste relevant context from `session.md` into your conversation
3. At session end, update the files with what happened
4. The manual step is copy/paste — only Claude Code auto-loads this

---

### 5. Model Tiering (Partially Portable)

Claude Code lets you assign different models to different agents via frontmatter (`model: opus`, `model: sonnet`, `model: haiku`).

**Other tools:** Most let you select a model globally but not per-task. The *principle* still applies:
- Use the strongest model for architecture and planning
- Use a balanced model for implementation and review
- Use the cheapest model for documentation

**Codex CLI** supports per-config model overrides, making it the closest equivalent.

---

### 6. Path-Scoped Rules (Most Tools Support This)

Loading different instructions for different parts of the codebase.

| Tool | Mechanism |
|------|-----------|
| **Claude Code** | `paths:` frontmatter in `.claude/rules/*.md` |
| **Cursor** | `globs:` in `.cursor/rules/*.mdc` |
| **Codex CLI** | Directory walk — `AGENTS.md` in any subdirectory scopes to that tree |
| **Gemini CLI** | Directory walk — `GEMINI.md` in subdirectories scopes automatically |
| **Windsurf** | `trigger: glob` in `.windsurf/rules/*.md` frontmatter |

---

## The Universal Takeaways

Regardless of which tool you use, these principles from the blueprint apply everywhere:

1. **Write your rules down.** Every tool has a rules file. Use it.
2. **Enforce, don't suggest.** Use hooks (Claude Code, Cursor), git hooks, or CI for critical rules.
3. **Scope your context.** Don't load everything into every session. Organize by domain.
4. **Verify outputs.** Never trust "done" without checking the actual result. Tool-agnostic.
5. **Track decisions.** An append-only decision log prevents "why did we do this?" across sessions.
6. **Match model to task.** Use expensive models for hard problems, cheap models for routine work.
7. **Separate enforcement from guidance.** Things that MUST happen need hooks/CI. Things that SHOULD happen go in rules files.

---

*This guide reflects the state of these tools as of early 2026. AI coding tools evolve rapidly — features marked "experimental" may stabilize or change.*
