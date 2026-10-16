---
name: cc-queued-prompt-hooks
description: "a prompt typed while a turn runs fires `UserPromptSubmit` at once, before that turn's `Stop`, carrying the running turn's `prompt_id` (CC 2.1.301); Esc'd or idle prompts get a fresh id; Ctrl+S stash fires nothing; mod `command.run`/`prompt.submit` leave composer text alone"
metadata:
  node_type: memory
  type: reference
  originSessionId: b7e0e424-2591-4c81-ba52-dff35a7dd1a5
  modified: 2026-10-16T15:57:56.044Z
---

Verified 2026-10-16 against Claude Code 2.1.301 in tmux, with hook loggers
(reports: `plans/reports/2026-10-16-queued-prompt-sweep-repro.md`,
`…-stashed-prompt-sweep.md`, `…-drive-with-composer-text.md`).

- **Queued prose fires `UserPromptSubmit` at queue time**, before the running
  turn's `Stop` and `turn.complete`. Typed mid-tool it is folded into the
  running turn (one `Stop` for both); typed during the final reply it becomes
  its own turn, but its hook still precedes the first turn's `Stop`.
- **`prompt_id` tells them apart without a lock.** A queued prompt's hook input
  carries the running turn's `prompt_id`. A prompt typed at an idle composer,
  or after an Esc (which fires no `Stop`), carries a fresh one. No other field
  differs. `PreToolUse` input carries the same `prompt_id` as its turn's
  `UserPromptSubmit`. Undocumented: treat an input without it as unknown.
- **Ctrl+S (`chat:stash`) fires nothing**, and the stash is not restored at
  turn end. Pulling a queued message back with Up fires no second hook.
- **A mod's `$.command.run` (`rename`, `clear`, `compact`) and
  `$.prompt.submit({asUser: true})` run over composer text and leave it
  unchanged**; the submit sends only the mod's text.
