---
name: project-precompact-drive
description: "SHIPPED + live-exercised 2026-07-20 (7f3c70c..1b6214a, unreleased); precompact drives commit-memory→compact→continue via Stop + SessionStart(compact); task file shared with handoff carries the content, prompt is only a handle"
metadata:
  node_type: memory
  type: project
  originSessionId: 8070a865-ea66-486b-9d50-a946add0a54c
  modified: 2026-07-20T19:06:05.046Z
---

precompact redefined from passive "flush memory, you `/compact`" to active
**commit memory → compact → continue**. Full design:
`docs/2026-07-18-precompact-drive-design.md`.

**Status (2026-07-20):** SHIPPED on `main`, `7f3c70c..1b6214a`, 101 bats + 9
pytest green. NOT yet released — `v0.9.0` is still the last tag; the release
covers this plus two earlier unreleased commits, and `just release minor` is
blocked by the auto-mode classifier so the user must run it.

**First live run** exercised the whole chain: `Stop` armed, the watcher typed
and submitted `/compact`, `SessionStart(compact)` fired the continuation. Two
defects it exposed, both now fixed (`4300dea`) — the line-2 Enter was absorbed
as a newline for want of a settle delay, and `is_typing` verification reported
that non-submit as success. Details in [[reference_cc_tui_tmux_driving]]. The
watcher fix and the shared-task-file change are themselves **not yet live-
tested**.

Shape: skill body stays vocabulary-free (run `handoff-precompact-probe`,
follow its directives — no gitlore/SDD vocab). ONE probe composes the gitlore
memory-commit directive + the SDD-ledger nudge via a new `_probe-lib.sh`
(shared with handoff's `handoff-memory-probe`). Memory commits through
gitlore's **file-trigger** (`.claude/gitlore-memory-message` +
`.claude/gitlore-commit-memory`, consumed by `memory-commit-batch.sh` on
PostToolBatch), NOT `commit-memory.sh -F -`. Compaction driven by a magic file
`.claude/autocompact` (exactly two lines: `/compact [directive]` /
single-line continuation), consumed by THREE hooks (revised 2026-07-19 after a
TUI spike): `write-compact.sh` PostToolUse **validates only** (systemMessage +
additionalContext on malformed, no spawn, no delete); `stop-compact.sh` on
**Stop** renames the file to `.pending` and spawns the line-1 watcher;
`load-compact.sh` on **SessionStart(compact)** spawns the line-2 watcher.
Watchers reuse `_rename-lib.sh`, do type-verify-submit for line 1, and read
only the visible pane. NOT a PostToolUse-spawned watcher polling for idle —
prose typed mid-turn is injected into the running turn, so screen-scraped
idleness is not a safe gate ([[reference_cc_tui_tmux_driving]]).
**The seam (2026-07-20):** both skills write `.claude/handoff-task.md`, and
`load-compact.sh` injects it at `SessionStart(compact)` via a shared
`handoff_frame` helper. The task file carries anything that must survive
verbatim; the continuation prompt is only a handle to it. This replaced an
earlier ban on precompact writing the task file — with no durable channel,
verbatim-critical facts got crammed into the one-line prompt. Consequences:
`handoff_activated()` accepts either skill (the guards gate on it), the frame
header is `# Task` not `# Handoff`, and precompact now leaves durable
cross-session state that is re-injected again at the next `startup|clear`.

Human-in-the-loop: the ONLY interactive pause is the memory-commit approval,
and only when memory is dirty; the compact directive + continuation are
informed-not-gated (invoking precompact authorizes the compaction).

**Why:** long/attended sessions need to compact and keep going without losing
durable memory to the summariser at full fidelity; the file-trigger sidesteps
the sandbox + auto-mode-classifier fragility of an agent-run git commit.

**How to apply:** implemented per the spec with inline TDD (per
[[project_worktree_handoff_root]] convention, not subagents). Two deviations
from the spec, both deliberate: the IPC directory resolves from the already-known
git root rather than `rev-parse --show-superproject-working-tree` (same answer
by construction, no extra subprocess, no empty-result edge case); and a
malformed `autocompact` surviving to `Stop` is consumed there rather than left
in place, so the same complaint does not repeat at every later `Stop`.

The TUI-driving facts the watchers depend on: [[reference_cc_tui_tmux_driving]].
Plugin skill bodies and `hooks.json` are snapshotted at session start, so
editing them mid-session does not activate them — see
[[reference_plugin_reload_session_start]].
