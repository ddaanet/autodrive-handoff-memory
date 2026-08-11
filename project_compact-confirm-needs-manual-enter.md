---
name: compact-confirm-needs-manual-enter
description: "the walker's line submission doesn't always land on its own, seen on `/compact` and `/rename`; open bug, fix later"
metadata: 
  node_type: memory
  type: project
  originSessionId: 1228fd49-f82a-49ea-98e6-bf3c011f0e9d
  modified: 2026-08-11T19:48:59.707Z
---

The walker's line does not always land reliably on its own — first seen on
a `/compact` line, a `UserPromptSubmit` failure report that session read:
"`/compact` did not land in the composer — a dead pane or copy mode may
have swallowed it. The transition or continuation it was driving did not
happen." My human partner had to press Enter manually to get it to submit.
A later session saw the identical failure report twice for `/rename` lines
(`handoff`'s pre-`/clear` rename), so the bug is not `/compact`-specific —
it is the walker's generic line-submission path.

**Why:** observed directly, cause not yet diagnosed — root cause could be in
`_submit_until`'s retry/confirm timing or in pane state generally
(`_watcher_lib.py`/`drive_when_idle.py`, formerly `_watcher-lib.sh`/
`drive-when-idle.sh`). See [[cc-tui-tmux-driving]] for the wider walker
confirmation mechanism this bug lives inside.

**How to apply:** until fixed, treat any armed walker-driven line (not just
`/compact`) as possibly needing a manual Enter as a working fallback —
don't assume silence past the failure-report window means the line was
typed hands-off. This is a fix-later item, not yet scheduled as a dated
design change.
