<!--
  Three session-start mechanisms exist by design (defense-in-depth):
  1. session-start hook — injects pre-computed workspace facts before Claude responds
  2. This rule — reads 3-4 core files during Claude's first response
  3. load-session skill — comprehensive 8-item restore with formatted summary
  The hook and rule are safety nets; the skill is the full restoration.
-->

> **SETUP REQUIRED:** Replace `{MEMORYCORE_PATH}` and `{CLAUDE_CONFIG_PATH}` with your actual paths before using this rule. If you skip this step, Claude will attempt to read file paths containing literal curly braces (e.g., `{MEMORYCORE_PATH}/core/session.md`), which will produce confusing "file not found" errors. See [skills/README.md](../skills/README.md#required-replace-placeholder-variables) for platform-specific examples.

# Session Lifecycle — Automatic Behaviors

These rules apply to EVERY session, regardless of project.

## Session Start (Before First Response)

When a new conversation begins (no prior messages in this session):

1. **Read session context** from `{MEMORYCORE_PATH}/core/session.md`
2. **Read user preferences** from `{MEMORYCORE_PATH}/core/preferences.md`
3. **Read active reminders** from `{MEMORYCORE_PATH}/core/reminders.md` — these may have deadlines
4. **Read architectural decisions** from `{MEMORYCORE_PATH}/core/decisions.md`
5. Use the loaded context to understand where the last session left off
6. If the user's first message implies continuation ("continue", "what were we doing", or jumping straight into a task from last session), reference the session recap naturally

Do NOT announce that you loaded memory. Just use it seamlessly.

## Session End (Natural Conversation Cues)

When the user signals they're done — phrases like "bye", "goodbye", "done", "that's all", "see you", "wrapping up", "signing off", "good night", "cya", "talk later", "finished for now":

1. **Update** `{MEMORYCORE_PATH}/core/session.md` with:
   - What was accomplished this session
   - Current working state
   - Next steps for the next session
2. **Update reminders** at `{MEMORYCORE_PATH}/core/reminders.md` — move completed to Completed section, add new ones
3. **Update per-project context**: If working on a registered project, update `{MEMORYCORE_PATH}/projects/active/{project}.md` → Session Context section
4. **If the session was significant** (feature shipped, bug fixed, architecture decision, new project started): append a diary entry to `{MEMORYCORE_PATH}/diary/current/YYYY-MM-DD.md`
5. **If new technical learnings or gotchas** were discovered: update `{CLAUDE_CONFIG_PATH}/projects/{project}/memory/MEMORY.md`
6. **Git commit + push** all MemoryCore changes (adjust remote and branch name to match your memory repo's configuration)
7. Move any next-steps that have persisted across 3+ sessions to `reminders.md` — session.md is for ephemeral context, not persistent deferrals

## Key Behavioral Notes

- The user does NOT use slash commands — detect intent from natural language
- Keep responses concise — no filler phrases, no emojis unless requested
- Prefer action over discussion — explain briefly, then do the thing
