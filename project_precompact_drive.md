---
name: project-precompact-drive
description: "RELEASED in v0.10.0 (2026-07-21), three live runs green; commit-memory→compact→continue via Stop + SessionStart(compact); known gap: submit_or_fail false-fails on queued submissions; \"/compact posted twice\" is CC's own double-record, not the plugin"
metadata:
  node_type: memory
  type: project
  originSessionId: 8070a865-ea66-486b-9d50-a946add0a54c
  modified: 2026-07-21T21:24:21.191Z
---

precompact redefined from passive "flush memory, you `/compact`" to active
**commit memory → compact → continue**. Full design:
`docs/2026-07-18-precompact-drive-design.md`.

**Status (2026-07-21):** RELEASED in `v0.10.0` (tag + GH release + marketplace
bump), alongside the whole-plugin simplify consolidation. Follow-up `v0.10.1`
carries the precompact task-file reset (both wipe hooks now match precompact).
`just release` is not categorically blocked by the auto-mode classifier — the
v0.10.0 run was, but `just release patch` went through unprompted when the
command description named both `handoff` and the `claude-plugins` marketplace
up front, per [[feedback-release-flow-naming]].

**First live run** exercised the whole chain: `Stop` armed, the watcher typed
and submitted `/compact`, `SessionStart(compact)` fired the continuation. Two
defects it exposed, both now fixed (`4300dea`) — the line-2 Enter was absorbed
as a newline for want of a settle delay, and `is_typing` verification reported
that non-submit as success. Details in [[reference_cc_tui_tmux_driving]].

**Second live run (2026-07-20) is green end to end**, with no keystroke from
the user: line 1 typed and submitted, `autocompact` → `.pending` → consumed,
the task-file frame injected at `SessionStart(compact)` (header `# Task —
<stamp>`), the write guard passing precompact's own task-file write, and line
2 auto-submitted.

**Third live run (2026-07-20 20:52, session 8e39f620)** delivered both lines
but exposed two report-level findings. (1) `submit_or_fail`'s `is_busy`
confirmation is a **false negative for queued submissions**: the continuation
Enter landed while the session was between compaction-complete and readiness,
so the prompt was *enqueued* (JSONL `queue-operation` enqueue → dequeue →
`promptSource:"queued"`), the pane never showed a spinner in the 3×0.5s
confirm window, and the watcher wrote "three Enters did not submit it" even
though the line was delivered. Fail wording overclaims; confirmation needs a
queued-aware signal (e.g. harness-side: grep the transcript tail for the
line, path available to the spawning hook as `transcript_path`). (2) The
"/compact posted twice" report is **not a plugin defect**: every `/compact` —
manual ones included, verified in sessions predating precompact — logs a RAW
prompt entry plus a `<command-name>` record ~30ms apart, and both render;
one submission, double-recorded by Claude Code itself.

**Watcher failures are now reported**, added on the back of that run rather
than skipped as speculative: a clean run is no evidence of observability, and
the failing run is the one nobody is watching. Watchers write a reason to
`.claude/autocompact.failed` (path handed down as `HANDOFF_FAIL_FILE` by the
spawning hook) and `report-compact-failure.sh` surfaces it at the next
`UserPromptSubmit` — earlier than `Stop`, since a watcher runs *after* the Stop
that spawned it. The dangerous path it covers is line 1's recognition abort:
`C-u` wipes the composer, so no compaction happens and the pane looks
untouched. Reports only what a watcher observed; a stale `.pending` is never
inferred as failure (legitimate for the whole Stop → compaction window).

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
