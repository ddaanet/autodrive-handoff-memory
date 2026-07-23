---
name: project-handoff-todo-ledger
description: "`.claude/handoff-todo.md` holds the remainder of a task list (open items only), git-tracked like the task file, both skills write it, wiped at activation, injected in the same frame as the task file, suppressed by a workflow ledger like SDD's"
metadata: 
  node_type: memory
  type: project
  originSessionId: 4cfddafb-441d-4f27-b514-ed3622e8fc11
  modified: 2026-07-23T14:59:23.058Z
---

Built 2026-07-22, from the complaint that `handoff-task.md` was being abused to
carry full task lists while the same items were tracked again in memory — so
completing one task meant edits in several places, each behind an approval gate.

The fix is a place, not a prohibition: `.claude/handoff-todo.md`, holding the
**remainder** of an active task list.

**Design points that carry weight.** Memory is not the place, structurally: one
file holds one fact, and the `project` type is scoped to work "not derivable
from code or git history", which the done half of a list is. The open half
clears the residual bar for the reason in
[[feedback_reconstructable_two_categories]]. Nothing names a todo tool, per
[[reference_todo_tool_flag_gated]] — the file is the ledger, any live tracker is
a cache, and the cache may legitimately hold more (item identity, ordering)
because that surplus is boundary-local.

**Shape.** Open items only; a finished item is dropped, never ticked. Injected
by `handoff_frame` after the task file under one `# Task —` header — either
file alone assembles a frame. Both skills write it in their existing parallel
batch. Force-added by `write-stage.sh` on the same terms as the task file
(2026-07-23 reversal — it shipped gitignored; see below). Wiped at activation, guarded by
read-guard/write-guard on the same terms as the task file — which made
`handoff_match_target` variadic over (basename, rel) pairs, since the jq parse
is on the Write/Edit hot path. `probe_ledger_path` in `_probe-lib.sh` is the
one-row registry: when superpowers SDD's `progress.md` exists it is the ledger
and the todo file stands down, emitted by precompact's nudge and by
`probe_todo_suppression` on the handoff path.

**Two decisions that reversed mid-design, both worth remembering.** precompact
*does* write it: the "tracker state survives compaction" argument only holds
when a tracker exists, and with none the list is context-resident — exactly what
the summariser paraphrases. And it *is* wiped despite being input carried
forward: both loaders re-inject it so re-authoring is from context, and without
the wipe a finished list lingers on disk forever and keeps re-injecting done
items as outstanding.

**Rejected:** firing the load at first `UserPromptSubmit` to put a new user task
ahead of the injected list. The hazard is real (a session once read injected
content as its own memory) but the fix that worked there was register and
volume, not position; and "first UPS" needs the armed/disarmed latch this plugin
rejects, on a per-turn hook. Also rejected: a probe deciding whether a list
exists — the agent knows for free, and a probe branch costs the
single-parallel-turn requirement a detection round-trip.

**Reversed 2026-07-23: it is versioned.** It shipped gitignored on two
arguments that do not survive being stated against the flow. The pairing with a
gitlore memory commit is about a *moment*, and one skill invocation writes both
files and commits the memory — both halves share it or neither does. The churn
was measured under gitlore's per-commit approval gate; in the main repo the todo
file moves exactly when the task file moves, a cost already accepted. Positively:
`## Current task` and `## Remaining` are two sections of one snapshot, and
tracking one puts half a frame in history — the half that names work not yet
done. It is a snapshot, not a mutating ledger: rewritten whole at a boundary.

Full rationale: DESIGN.md, "A place for the todo list (2026-07-22)" and
"Overflow deserves the same persistence (2026-07-23)".
