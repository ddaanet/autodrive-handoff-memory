---
name: project-precompact-drive
description: "commit-memory→compact→continue via Stop + SessionStart(compact); neither driven line confirms on is_busy any more — line 2 keys on the transcript, line 1 on SessionStart(compact) consuming .pending; \"/compact posted twice\" is CC's own double-record, not the plugin"
metadata:
  node_type: memory
  type: project
  originSessionId: 8070a865-ea66-486b-9d50-a946add0a54c
  modified: 2026-07-24T17:16:15.631Z
---

precompact redefined from passive "flush memory, you `/compact`" to active
**commit memory → compact → continue**. Full design:
`docs/2026-07-18-precompact-drive-design.md`.

Shipped and live-exercised; released alongside the whole-plugin simplify
consolidation. Version and release-status bookkeeping is deliberately not
tracked here — it rots silently and `git log` answers it, per
[[feedback_memory_status_claims_rot]].

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
though the line was delivered. **Fixed:**
line-2 confirmation moved off `is_busy` to the transcript. `submit_confirmed_or_fail` +
`transcript_prompt_count` (`_rename-lib.sh`) baseline the count of genuine
user-prompt entries containing the typed line, Enter, and poll for it to rise;
`load-compact.sh` exports the session `transcript_path` as `$HANDOFF_TRANSCRIPT`.
The count keys on the same structural flags as `handoff_activated` (excludes the
isMeta frame / isCompactSummary echoes) and is baselined pre-Enter, so neither a
quoted copy in the summary nor a stale pre-compaction entry reads as a submit.
Line 1 kept `is_busy` on the reasoning that its `/compact` at Stop starts a
reliably-busy compaction, never queued — **falsified 2026-07-22**, see below.
Root cause verified against the 8e39f620 transcript
(queued entry logged at 20:54:06, turn spinner at 20:54:19 — 13s later, past the
window). (2) The
"/compact posted twice" report is **not a plugin defect**: every `/compact` —
manual ones included, verified in sessions predating precompact — logs a RAW
prompt entry plus a `<command-name>` record ~30ms apart, and both render;
one submission, double-recorded by Claude Code itself.

**Watcher failures are now reported**, added on the back of that run rather
than skipped as speculative: a clean run is no evidence of observability, and
the failing run is the one nobody is watching. Each watcher writes a reason to
a file named for the line it drove (`.claude/autocompact.failed`,
`.claude/autorename.failed`; path handed down as `HANDOFF_FAIL_FILE` by the
spawning hook) and `report-watcher-failure.sh` surfaces them at the next
`UserPromptSubmit` — earlier than `Stop`, since a watcher runs *after* the Stop
that spawned it. The dangerous path it covers is line 1's recognition abort:
`C-u` wipes the composer, so no compaction happens and the pane looks
untouched. Reports only what a watcher observed; a stale `.pending` is never
inferred as failure (legitimate for the whole Stop → compaction window).

**Fourth live run (2026-07-22) delivered both lines and still reported a
failure**, which is what falsified line 1's `is_busy`: the `/compact` submitted
at 20:06:46, compacted for 103s, and the watcher wrote "three Enters did not
submit it" anyway. Whatever the TUI renders in the ~1.5s after that keystroke,
`is_busy` does not match it. The transcript trick does not transfer either — the
`/compact` entry carries the submit timestamp but is *persisted* only when the
command finishes, landing in the JSONL physically after the compact summary
written 103s later. So line 1 stops asking whether the keystroke landed and asks
whether the compaction happened: `stop-compact.sh` hands the watcher the
`.pending` path as `HANDOFF_PENDING_FILE` and `submit_consumed_or_fail` waits
for `SessionStart(compact)` to consume it. Slower to report a real failure, and
free of false ones. Same run also fixed a `.claude/autocompact` that outlives an
abnormally-ended turn (Esc, crash, quit) and gets armed by a later, unrelated
`Stop`: `UserPromptSubmit` cannot fire between the write and that turn's own
`Stop`, so anything it can see never armed and is swept.

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

**Fifth live run (2026-07-24) reported a false line-2 failure again**, a
regression on the fix from the third run above. Root cause this time:
`submit_confirmed_or_fail`'s confirm window was 3×`VERIFY_DELAY` (~1.5s)
total, but the observed gap between the compaction's summary being ready and
`compact_boundary` actually landing in the transcript was ~18s — the Enters
registered and the prompt did arrive, just long after the watcher had already
given up and recorded a failure. **Fixed:** `submit_confirmed_or_fail` now
matches `submit_consumed_or_fail`'s two-phase shape — 3 fast retries (Enter +
check, covers an Enter that never registered), then a long poll up to
`CONSUME_TIMEOUT` **without** resending Enter (avoids a duplicate submit)
while waiting for a delayed transcript write. Test coverage in
`tests/rename-test.bats` reproduces the delayed-landing case with an
async background writer, timed past the fast-retry phase — which itself
costs real time, since each `transcript_prompt_count` check spawns a
python3 subprocess (~60ms/call measured), so a naive short synthetic delay
in a test can land inside the fast-retry phase by accident rather than
after it.
