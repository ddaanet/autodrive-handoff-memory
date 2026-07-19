---
name: project-precompact-drive
description: "precompact redesign — commit memory → compact → continue, driven by a tmux watcher; approved spec at docs/2026-07-18-precompact-drive-design.md; migrates handoff's memory commit to gitlore's file-trigger"
metadata:
  node_type: memory
  type: project
  originSessionId: 8070a865-ea66-486b-9d50-a946add0a54c
---

precompact is being redefined from passive "flush memory, you `/compact`" to
active **commit memory → compact → continue**. Full approved design:
`docs/2026-07-18-precompact-drive-design.md` (brainstormed + verified
2026-07-18; NOT yet implemented).

Shape: skill body stays vocabulary-free (run `handoff-precompact-probe`,
follow its directives — no gitlore/SDD vocab). ONE probe composes the gitlore
memory-commit directive + the SDD-ledger nudge via a new `_probe-lib.sh`
(shared with handoff's `handoff-memory-probe`). Memory commits through
gitlore's **file-trigger** (`.claude/gitlore-memory-message` +
`.claude/gitlore-commit-memory`, consumed by `memory-commit-batch.sh` on
PostToolBatch), NOT `commit-memory.sh -F -`. Compaction driven by a magic file
`.claude/autocompact` (exactly two lines: `/compact [directive]` /
single-line continuation) → new `write-compact.sh` PostToolUse hook → detached
`compact-when-idle.sh` watcher (mirrors autorename / `rename-when-idle.sh`,
reuses `_rename-lib.sh`), with line 2 gated on the `⎿ Compacted (` marker.
Human-in-the-loop: the ONLY interactive pause is the memory-commit approval,
and only when memory is dirty; the compact directive + continuation are
informed-not-gated (invoking precompact authorizes the compaction).

**Why:** long/attended sessions need to compact and keep going without losing
durable memory to the summariser at full fidelity; the file-trigger sidesteps
the sandbox + auto-mode-classifier fragility of an agent-run git commit.

**How to apply:** implement per the spec with inline TDD (per
[[project_worktree_handoff_root]] convention, not subagents). Migrating
`memory-probe.sh` to the file-trigger also closes a standing gap — gitlore
built the file-trigger FOR handoff (handoff-only, settled 2026-07-17) but
handoff still emits `-F -`. The TUI-driving facts the watcher depends on:
[[reference_cc_tui_tmux_driving]]. Update DESIGN.md in the same pass
([[feedback_update_design_doc]]).
