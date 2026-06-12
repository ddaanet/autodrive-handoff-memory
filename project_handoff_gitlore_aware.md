---
name: project-handoff-gitlore-aware
description: "handoff proactively commits gitlore memory at wrap-up; gitlore prerequisite SHIPPED, spec+plan written, ready to implement"
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

**Status (2026-06-12):**
- gitlore prerequisite **SHIPPED**: `commit-memory.sh` (D16), arg-driven
  (`-m`/`-F -`), commits the submodule + advances local `live` with no parent
  commit. Discover via `git config gitlore.commitCommand` (self-healing,
  re-pinned every SessionStart). Activation gate = `gitlore-memory` submodule
  in `.gitmodules` (FR12). Runs without `CLAUDE_PLUGIN_ROOT` (falls back to
  `gitlore.hooksDir`).
- handoff spec + plan **written**:
  `docs/superpowers/specs/2026-06-12-gitlore-aware-handoff-design.md`,
  `docs/superpowers/plans/2026-06-12-gitlore-aware-handoff.md`. 7 TDD tasks.
  Next session: execute the plan (subagent-driven or inline).

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
