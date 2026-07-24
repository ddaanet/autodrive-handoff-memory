---
name: handoff-tree-local
description: Handoff files stay under the project worktree's `.claude/` — never relocated into the shared gitlore memory submodule; `write-stage.sh`'s `git add -f` versions them in the local repo
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 9d88f92e-9300-43ce-97d7-d0bf39950dc6
  modified: 2026-07-24T19:13:33.699Z
---

`.claude/handoff-task.md` and `.claude/handoff-todo.md` stay under `.claude/` in the project worktree. Do not relocate them into the gitlore memory submodule.

**Why:** A handoff captures in-progress task state for one checkout — it is tree-local, and it is ephemeral by design (both files are wiped at the next activation). Gitlore memory is a shared remote submodule: putting handoff files there would publish one worktree's transient task state to every clone, and would outlive the task it describes. The two stores answer different questions — gitlore holds durable facts and decisions, the handoff frame holds what this checkout was in the middle of.

**How to apply:** Keep the split at the boundary of durability. A fact worth carrying past the task goes to gitlore memory; the task snapshot itself stays local. For versioning the handoff files, `write-stage.sh` staging them with `git add -f` is the right mechanism — they are `.gitignore`d, so only that hook adds them, and they land in the local project repo rather than a shared submodule. Hooks anchor on `handoff_root` (the enclosing worktree root), so a worktree session versions its own `.claude/`, not the main tree's.

See [[feedback-gitlore-memory]] for the converse rule — memory/ is tracked and commits with the code, never gitignored.
