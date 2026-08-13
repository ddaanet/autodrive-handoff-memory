---
name: pane-fixtures-are-captures
description: "editing a walker pane predicate/fixture, or a driven line not landing: `just tui-conformance` is the only check against a real TUI"
metadata: 
  node_type: memory
  type: project
  originSessionId: f9d05d38-44c5-4dc6-b0de-23fe83154129
  modified: 2026-08-13T16:12:52.699Z
---

The walker's pane predicates in `scripts/_watcher_lib.py` are checked against
a **live** Claude Code TUI by `just tui-conformance` (pytest `live` marker,
deselected from `just precommit`). It needs a real `claude` and a real tmux
server, so it cannot run sandboxed; it submits nothing and costs no tokens.
Run it whenever a pane predicate or its fixtures change, and whenever a driven
line stops landing. The frozen strings in `tests/test_watcher_lib.py` are
**captures**, never hand-written — the `NBSP` constant is the composer gap the
TUI actually paints.

**Why:** every hand-written fixture had an ASCII space where the TUI pads with
U+00A0, so `is_typing` was False for every line the walker ever typed while
its suite stayed green — the predicate was tested against its own premise. The
probe found three more defects on its first runs: the idle composer is never
empty (a faint placeholder), the "No commands match" text lands 1–2s after the
send, and the composer does not always repaint within `VERIFY_DELAY`.

**How to apply:** never hand-write a pane fixture, and never fix a pane
predicate by reasoning about what the chrome probably looks like — capture it.
A predicate read in two polarities (confirm *and* bail) inverts silently in
both directions at once, so widening one can arm the other; check both. See
`docs/changelog/2026-08-13-predicates-match-the-real-pane.md`, and
[[green-is-not-evidence]] for the wider class.
