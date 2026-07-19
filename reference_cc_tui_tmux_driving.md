---
name: reference-cc-tui-tmux-driving
description: "Driving the Claude Code TUI via tmux send-keys — input during busy queues (not interrupts), readiness ≠ ❯ glyph, busy = (Ns · timer, /compact submits via send-keys, compaction-done = ⎿ Compacted ( marker (verified CC v2.1.214)"
metadata:
  node_type: memory
  type: reference
  originSessionId: 8070a865-ea66-486b-9d50-a946add0a54c
---

Verified 2026-07-18 against Claude Code v2.1.214 (Haiku 4.5, tmux 3.5a) by
driving a throwaway `claude` session in a private tmux socket (capture-pane +
send-keys), for the handoff plugin's compaction-driver watcher.

- **Input during any busy op is queued, not interrupted or merged.** Typing +
  Enter while a turn OR compaction runs → "Press up to edit queued messages";
  the text runs as its own turn afterward. Esc interrupts; plain typing does
  not.
- **Readiness ≠ the `❯` glyph.** The prompt glyph shows while the backend is
  still `/rc connecting…`; keystrokes/Enter sent then do NOT submit. Wait for
  stable connected-idle (what `rename-when-idle.sh` already does), not the
  glyph.
- **Busy indicator = the `(<n>s ·` timer + a randomized gerund**
  (`Frolicking…`, `Fiddle-faddling…`, `Effecting…` — the word varies, the
  timer doesn't). `_rename-lib.sh`'s `is_busy` grep `\([0-9]+s ·|esc to
  interrupt` still matches via the timer clause on this version.
- **`/compact <directive>` submits via `send-keys -l` + `Enter`** and triggers
  real compaction: `⎿ Compacted (ctrl+o to see full summary)` marker appears
  and `SessionStart:compact` hooks fire.
- **Compaction can be too fast to catch as "busy"** (~40k Haiku context
  compacted between 0.2s polls). Use the `⎿ Compacted (` marker as the
  completion signal, NOT a busy→idle transition.
- **A continuation sent after the marker runs against the compacted context**
  (verified by a correct answer sourced from the retained summary).

Empirical basis for [[project_precompact_drive]]; the watcher's reliability
design rests on these. Sibling of [[feedback_tmux_cc_popup]].
