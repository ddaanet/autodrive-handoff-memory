---
name: project-autoname-removed
description: autoname plugin removed entirely; handoff is the sole session-rename path
metadata: 
  node_type: memory
  type: project
  originSessionId: d682c4a2-149f-4b7d-a03e-317253062c72
---

The `autoname@ddaanet` plugin has been removed: uninstalled from settings, GitHub repo deleted, marketplace entry removed, local repo deleted.

**Why:** Running both handoff's `write-rename.sh` hook and the `autoname` skill in the same H expansion spawned two `rename-when-idle.sh` watchers concurrently, causing them to interleave `send-keys` calls and produce a corrupted session name.

**How to apply:** Session renaming now happens exclusively via handoff's `autorename` file → `write-rename.sh` hook → `rename-when-idle.sh` watcher. Do not suggest reinstalling or recreating the autoname plugin. The H expansion no longer includes `/autoname`.
