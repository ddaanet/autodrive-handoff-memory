---
name: cc-tui-tmux-driving
description: "driving a *running* TUI: mid-turn input as TUI-local / harness-action queued / plugin-command own-turn / prose injected into the running turn; `Stop` + `SessionStart(compact)` are the authoritative gates (`Stop` doesn't fire on Esc); never confirm a submit by reading the pane"
metadata:
  node_type: memory
  type: reference
  originSessionId: 8070a865-ea66-486b-9d50-a946add0a54c
  modified: 2026-09-21T15:50:34.637Z
---

Verified 2026-07-19 against Claude Code v2.1.215 (tmux 3.5a) by driving a
throwaway `claude` session in a detached tmux session, logging every hook.

**Mid-turn input has four classes**, not one "queued" behavior:

| typed mid-turn | behavior | UserPromptSubmit |
|---|---|---|
| `/focus` (TUI-local) | intercepted at once, never queued | — |
| `/compact` (harness action) | queued → run at turn boundary | no |
| `/handoff` (plugin/skill) | queued → drains as its own turn | **yes** |
| plain prose | **injected into the running turn's next call** | no |

- **Prose is the dangerous input** — it contaminates the running turn. Slash
  input never reaches the model as prose: unrecognized commands drain to
  `● Unknown command: …`.
- **Queued input does not bypass hooks in general.** `/compact` skips
  UserPromptSubmit only because it never becomes a prompt; a queued plugin
  command fires it with the raw command text as `prompt`.
- **Hook chain:** `Stop` → `PreCompact` (+0.75s) → `SessionStart`
  `"source":"compact"` (+47s). Queue drains ~0.75s after `Stop`.
- **`Stop` does not fire on Esc** — interrupted turns fail safe. Its payload
  carries `last_assistant_message`, `stop_hook_active`, `prompt_id`, `effort`,
  `background_tasks`, `session_crons`.
- **Command recognition is visible before Enter**: `/compact` renders an
  autocomplete row, `/comzzz` renders `No commands match "/comzzz"` — which is
  what makes type-verify-submit implementable.
- **Busy = the `[0-9]+s ·` timer** + a randomized gerund, read from the
  **visible** pane only: `capture-pane -S` history matches a stale timer and
  reports busy long after `Stop`.
- **An Enter right after a long `send-keys -l` is absorbed as a line break**,
  not a submit — it lands in the TUI's paste window. Always settle between the
  literal send and the first Enter.
- **Never confirm a submit by reading the pane.** Not "the composer looks
  empty" (an absorbed Enter leaves a multi-line composer that reads as empty),
  and not `is_busy` either: a queued submit is accepted and transcript-logged
  at once but shows no spinner until its turn starts, and a 103s compaction
  showed none. Confirm against something the harness owns — a file the
  transition's `SessionStart` consumes, or a new transcript entry.
- **A pane that is still booting answers none of the above**: the `❯` glyph is
  not readiness, and keystrokes typed across a boot are lost over a discrete
  band and merely unpainted after it — [[cc-boot-input-window]].
- **The tmux socket is unreachable from the agent's sandboxed Bash**
  (`Operation not permitted`) — hook-spawned watchers are the only
  sandbox-clean path; spike driving needs `dangerouslyDisableSandbox`.
- **`claude --plugin-dir <path>`** (repeatable) loads a dev plugin tree into a
  throwaway session; its commands register unnamespaced (`/handoff`).
- **`/compact` is double-recorded** — one submission, two transcript entries
  ~30ms apart (a RAW prompt plus a `<command-name>` record), manual ones
  included. Count structural signals, never entries.
- **`tmux display-popup` is invisible under iTerm2's `-CC` control mode** —
  use `tmux new-window`.
