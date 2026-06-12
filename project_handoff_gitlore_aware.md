---
name: project-handoff-gitlore-aware
description: "handoff proactively commits gitlore memory at wrap-up; SHIPPED + merged to main (commit 80a0574, 2026-06-12)"
metadata: 
  node_type: memory
  type: project
  originSessionId: 2374725e-5904-4dcf-b8f6-67e3f33aa929
---

Decision (2026-06-12): handoff becomes **gitlore-aware**. At wrap-up, when the
memory submodule is dirty, a read-only probe emits a directive and the agent
summarizes the changes → gets explicit user approval (prose, editable) →
commits via gitlore's blessed entry point.

**Why:** `/commit-commands:commit` forbids user interaction; gitlore mandates
per-commit approval (FR11). Coupling review+commit at the interactive handoff
moment avoids drift / redundant reviews.

**Scope:** commit ONLY memory. "push" = local `live` (`HEAD:live`) only, NOT
origin — origin push stays with gitlore's ambient `pre-push` hook.

**Status (2026-06-12): SHIPPED + merged to `main`** (fast-forward to commit
`80a0574`). Release not yet run — `just release` ships the minor bump to the
marketplace when wanted.
- gitlore prerequisite **SHIPPED**: `commit-memory.sh` (D16), arg-driven
  (`-m`/`-F -`), commits the submodule + advances local `live` with no parent
  commit. Discover via `git config gitlore.commitCommand` (self-healing,
  re-pinned every SessionStart). Activation gate = `gitlore-memory` submodule
  in `.gitmodules` (FR12). Runs without `CLAUDE_PLUGIN_ROOT` (falls back to
  `gitlore.hooksDir`).
- handoff implementation **landed**: `scripts/memory-probe.sh` +
  `bin/handoff-memory-probe` + `tests/memory-probe.bats` (7 cases) wired into
  the justfile; SKILL.md Step 2/3 wiring; non-tmux rename `additionalContext`
  fix; docs. Built subagent-driven (spec + code-quality review per task,
  final holistic review).
- **Plan was critically simplified 7→3 tasks before execution** (probe+shim+
  wiring as one coupled TDD unit; rename fix; skill+docs; verification inline
  not dispatched) — applied/validated [[feedback-subagents-over-applied]]: the
  axis is batching, not one-agent-per-task. The `/simplify-plan` user command
  was created to do this critique on future plans.

**Architecture resolved:** four-part split — detect (script) → summarize
(agent) → approve (human) → commit (gitlore script). New
`scripts/memory-probe.sh` owns the whole dirty-or-not conditional and prints
the agent's next action; the skill body just runs it and follows output (no
`if dirty` in agent — see [[feedback-harness-over-agent]]). Exposed to the
agent's Bash via `bin/handoff-memory-probe` (NOT a hook) — see
[[reference-skill-bundled-scripts]] for why. The probe couples only to the
`gitlore-memory` submodule registration and the `commitCommand` key.

**Decisions settled this session:**
- Probe is a PATH-shimmed script, not a hook: the agent must *act* on the
  result, and the conditional lives in the script regardless.
- Three-parallel batch confirmed: two Write-**tool** calls + one probe Bash
  call; no heredoc (would bypass `write-stage.sh` / `write-rename.sh`).
- Non-tmux autorename: route the `/rename` line through `additionalContext`
  (agent-facing), drop the dead "relay the systemMessage" line. See
  [[feedback-hook-agent-feedback]].
- gitlore's `commit-memory.sh` STAYS in gitlore `scripts/` (discovered via the
  key), NOT moved to `bin/`: the key is self-healing + no layout coupling;
  moving it reopens a shipped feature and widens its surface.
