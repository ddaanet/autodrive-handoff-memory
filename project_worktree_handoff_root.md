---
name: project-worktree-handoff-root
description: "Done (2026-06-09, commits d244e34..5d0145a) — handoff hooks now anchor on the enclosing git-worktree root via worktree_root.py so worktree sessions own their own .claude/; released in v0.9.0"
metadata: 
  node_type: memory
  type: project
  originSessionId: 7a891994-6ca1-4ea3-a503-1259d89a76e2
  modified: 2026-07-20T13:18:59.048Z
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

**Status (2026-06-09):** SHIPPED in 6 commits `d244e34..5d0145a` on `main`.
Executed **inline with TDD, not subagents** (see [[feedback-subagents-over-applied]])
— the plan's 6 commit boundaries were kept. Resolutions: empty-cwd guard **kept**
(`if not cwd: return project`); spec+plan **folded into** the Task 6 docs commit.
Full suite green (46 bats + 86 pytest). Released in **v0.9.0** (verified
2026-07-20: `git merge-base --is-ancestor 5d0145a v0.9.0` succeeds).

See [[feedback-claude-project-dir]], [[feedback-handoff-tree-local]].
