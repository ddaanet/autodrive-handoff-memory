---
name: compact-confirm-needs-manual-enter
description: "The drive-when-idle walker's automatic `/compact` submission is unreliable and sometimes needs a manual Enter press — open bug, not yet fixed"
metadata: 
  node_type: memory
  type: project
  originSessionId: 1228fd49-f82a-49ea-98e6-bf3c011f0e9d
  modified: 2026-08-10T17:12:38.124Z
---

The walker's `/compact` line does not always land reliably on its own — a
`UserPromptSubmit` failure report this session read: "`/compact` did not
land in the composer — a dead pane or copy mode may have swallowed it. The
transition or continuation it was driving did not happen." My human partner
had to press Enter manually to get it to submit.

**Why:** observed directly, cause not yet diagnosed — root cause could be in
`_submit_until`'s retry/confirm timing or in the compact-specific pane state
(`_watcher_lib.py`/`drive_when_idle.py`, formerly `_watcher-lib.sh`/
`drive-when-idle.sh`). See [[cc-tui-tmux-driving]] for the wider walker
confirmation mechanism this bug lives inside.

**How to apply:** until fixed, treat an armed `/compact` transition as
possibly needing a manual Enter as a working fallback — don't assume
silence past the failure-report window means the compaction ran hands-off.
This is a fix-later item, not yet scheduled as a dated design change.
