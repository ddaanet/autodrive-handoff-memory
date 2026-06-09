---
name: project-worktree-handoff-root
description: "Ongoing fix — anchor handoff hooks on the enclosing git-worktree root so worktree sessions own their own .claude/; design + plan written, execution pending"
metadata: 
  node_type: memory
  type: project
  originSessionId: 7a891994-6ca1-4ea3-a503-1259d89a76e2
---

**Bug:** in a git-worktree session the handoff hooks anchor on
`CLAUDE_PROJECT_DIR` (pinned to the main tree), so a worktree write to
`<wt>/.claude/handoff-task.md` is denied and the round-trip collides on the main
repo's single `.claude/`. Report: `plans/bug-worktree-handoff-location.md`.

**Chosen fix:** a pure resolver `scripts/worktree_root.py`
(`worktree_root(cwd, project)`) that walks up from the session `.cwd` via on-disk
`.git` linkage to the enclosing linked-worktree root, else returns
`CLAUDE_PROJECT_DIR`. A one-line `handoff_root` wrapper in `_lib.sh` shells out
to it; every cwd-scoped hook (write-guard, read-guard, write-stage,
load-handoff, _wipe-emit + skill/prompt pre-hooks, write-rename) routes through
it. Outside a worktree → byte-identical to today, so the existing suite stays
green. Direct port of cwd-safety's `_worktree_root`. Rejected: raw `.cwd` as root
(drift-prone) and recording the root via Worktree/CwdChanged hooks (observational
+ no clean per-worktree storage — see [[reference-hook-input-schema]]).

**Status (2026-06-09):** spec
`plans/2026-06-09-per-worktree-handoff-root-design.md` and 6-task TDD plan
`plans/2026-06-09-per-worktree-handoff-root-plan.md` written and self-reviewed.
Execution is **subagent-driven, next session** (user's choice).

**Open decisions to resolve at execution:** (1) keep vs drop the empty-cwd guard
in `worktree_root.py` — recommended keep (makes the resolver total +
CWD-independent; exercised by the existing cwd-less load-handoff bats tests);
(2) commit spec+plan standalone or fold into the Task 6 docs commit (plan does
the latter). See [[feedback-claude-project-dir]], [[feedback-handoff-tree-local]].
