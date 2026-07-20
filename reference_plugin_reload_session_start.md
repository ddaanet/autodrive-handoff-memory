---
name: reference-plugin-reload-session-start
description: Plugin skill bodies (and almost certainly hooks.json) are snapshotted at session start — editing a plugin mid-session does not activate the change; observed 2026-07-20 in Claude Code
metadata: 
  node_type: memory
  type: reference
  originSessionId: 94e0a066-a634-4a25-a32d-b0f53b992c25
  modified: 2026-07-20T17:40:52.507Z
---

Editing a plugin's own files mid-session does **not** make the change live in
that session. Observed directly 2026-07-20: after rewriting
`skills/precompact/SKILL.md` and committing it, invoking
`/handoff:precompact` in the same session injected the **pre-rewrite** body,
even though the skill's reported base directory was the working tree
(`/Users/david/code/handoff/skills/precompact`) and the file on disk held the
new text.

So the loaded content is snapshotted at session start, not read at invocation.
By the same mechanism `hooks/hooks.json` is near-certainly session-start-loaded
too — **inferred, not directly observed**: newly-declared hook events (a fresh
`Stop` entry, a widened `SessionStart` matcher) should be assumed inactive
until a new session. Verify before relying on it either way.

**Why it matters:** when developing a plugin *in* the session that uses it, a
new magic file can be written with no hook registered to consume it. That is a
silent no-op, not an error — nothing fires, nothing complains, and the failure
looks like the feature simply not working.

**How to apply:** when self-testing plugin changes mid-session, either restart
the session first or fall back to the previously-shipped path and say so. Never
report a mid-session plugin edit as "now active". Bundled *scripts* invoked by
bare-name `bin/` shim are the exception — PATH is set at session start but
points at the live directory, so script edits take effect immediately
([[reference_skill_bundled_scripts]]).
