---
name: project-handoff-gitlore-aware
description: "handoff will proactively commit memory during its flow once gitlore ships a standalone commit entry point; memory-only, push=live not origin"
metadata: 
  node_type: memory
  type: project
  originSessionId: 2374725e-5904-4dcf-b8f6-67e3f33aa929
---

Decision (2026-06-12): handoff becomes **gitlore-aware**. When the memory
submodule is active and dirty, handoff proactively summarizes the memory
changes, gets explicit user approval, writes the approved summary to the
`gitlore-commit-msg` file, and invokes a **new gitlore standalone entry point**
to commit memory + advance local `live`.

**Why:** `/commit-commands:commit` forbids user interaction; gitlore mandates
per-commit approval (FR11). Coupling review+commit at the interactive handoff
moment avoids drift / redundant reviews and keeps later non-interactive code
commits clean of the gate.

**Scope:** commit ONLY memory (not `handoff-task.md` or code). "push" = local
`live` branch (`HEAD:live`) only, NOT origin — origin push stays with gitlore's
ambient `pre-push` hook.

**Prerequisite (parked):** gitlore must first add the standalone entry point.
Brief: `/Users/david/code/gitlore/brief-standalone-memory-commit.md`. handoff's
own integration is deferred until that lands.

**Handoff-side open decisions** (settle when resuming):
- Keep the three-parallel pattern at write time (two Write-**tool** calls +
  one detection Bash call). Do NOT heredoc the file writes into one script —
  that bypasses `write-stage.sh` (stages `handoff-task.md` + saves the session
  pointer) and `write-rename.sh`. See [[feedback-harness-over-agent]].
- Non-tmux autorename: route the `/rename <title>` line through
  `additionalContext` (agent-facing) with an instruction to print it fenced,
  replacing the dead "relay the systemMessage" line in SKILL.md — agents never
  see `systemMessage`. See [[feedback-hook-agent-feedback]].

**Detection:** gitlore active = `gitlore_has_submodule` / `git config
gitlore.hooksDir` non-empty.
