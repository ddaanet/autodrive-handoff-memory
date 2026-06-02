---
name: handoff-tree-local
description: Handoff files must stay tree-local — do not relocate them to gitlore shared memory
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 9d88f92e-9300-43ce-97d7-d0bf39950dc6
---

Handoff files (`.claude/handoff-task.md`, `.claude/handoff.md`) must stay under `.claude/` in the project worktree. Do not relocate them to the gitlore memory root.

**Why:** The handoff captures in-progress task state for the current checkout — it is tree-local. Gitlore memory is a shared remote submodule; putting handoff files there would share ephemeral task state across all clones.

**How to apply:** If gitlore integration is ever revisited, limit it to things that genuinely belong in shared memory (durable facts, decisions). For versioning handoff files, `git add -f` in the extract hook is the right mechanism — it stages them into the local project repo, not a shared submodule.
