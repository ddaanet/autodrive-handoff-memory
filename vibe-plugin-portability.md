---
name: vibe-plugin-portability
description: "Vibe 3.0.1 exposes only `post_agent`/`pre_tool`/`post_tool` hooks (no SessionStart/UserPromptSubmit/PreCompact); the Claude adapter refuses mods and passes hook matchers through unrenamed; AGENTS.md splices verbatim (no `@` import), CLAUDE.md is never read, there is no native memory"
metadata:
  node_type: memory
  type: reference
  modified: 2026-11-23T14:03:33.103Z
---

Verified 2026-11-22 against mistral-vibe 3.0.1 (`vibe/core/hooks/models.py`,
`vibe/core/plugins/_claude.py`, `mistralai_vibe_local_harness/vibe/_foreign_hooks.py`,
`vibe/core/tools/builtins/skill.py`) and one headless `claude -p` probe.

- **Hook surface is three events**: `HookType = POST_AGENT | PRE_TOOL | POST_TOOL`.
  No SessionStart, no UserPromptSubmit, no PreCompact; `/compact` keeps the
  session id, `/clear` starts a new one.
- **Claude plugin adapter maps three events only** (`PreToolUse`→`pre_tool`,
  `PostToolUse`→`post_tool`, `Stop`→`post_agent`) and the rest are reported
  unsupported. Only synchronous shell command hooks, no `if` filter,
  no `async`. Foreign plugins contribute skills + MCP servers; executable
  `.js`/`.ts` is refused, so a Claude mod cannot run.
- **Claude hook matchers pass through no renaming**: the adapter compiles the
  raw regex, but Vibe's tool names differ (`Bash`→`bash`, `Write`/`Edit`→
  `write_file`/`edit`), so a matcher like `"Write|Edit"` never fires. Use
  `re:write_file|edit` when porting.
- **`post_tool` `hook_specific_output.additional_context` works**: the harness
  appends it to the tool result the model sees; `decision: "deny"` replaces
  the result text with `reason`. The hook stdin payload carries `session_id`,
  which is what lets an injector self-exclude the writing session.
- **`post_agent` deny injects the `reason` as a retry user message**, capped 3
  per hook per turn — not a prompt submission channel.
- **The skill tool prints `Base directory for this skill: <path>`** on load, so
  scripts shipped inside a skill directory are callable by absolute path;
  `PLUGIN_ROOT` is injected into MCP server processes only.
- **AGENTS.md is spliced verbatim** (`get_agents_md_section`) — no `@` import
  expansion, so file content crosses only by instructing the agent to read.
  CLAUDE.md is never read; the Claude-home adapter imports session history
  only.
- **No native memory system**: nothing in config, nothing in the harness; the
  out-of-tree store (gitlore) is the only durable carrier. `claude -p` refuses
  to reproduce its own memory prompt section, but the content is not needed —
  the standing write trigger and session-start read are two AGENTS.md lines.
