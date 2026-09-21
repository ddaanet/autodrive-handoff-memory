---
name: cc-boot-input-window
description: "when a booting or `--resume`d `claude` actually accepts keystrokes: the `❯` glyph is not readiness, gate on cursor-in-composer, and input loss is a discrete ~1–1.5s mid-boot band rather than everything before the first paint"
metadata: 
  node_type: memory
  type: reference
  originSessionId: ea59e561-3a20-44e1-bcad-bb94d9ce5d48
  modified: 2026-09-21T15:50:18.666Z
---

Driving a Claude Code TUI that is still starting — a `--resume` relaunch, a
fresh launch — against tmux. Measured on this box; the spreads track load, so
treat the shape as the fact and the numbers as one machine's. See
[[cc-tui-tmux-driving]] for everything about a session already running.

- **Readiness ≠ the `❯` glyph.** It shows while the backend is still
  `/rc connecting…`, a `--resume` replays conversation text that can contain
  one, and a freshly exec'd `claude` shows no busy chrome either — so an
  idle-poll gate reads a whole boot as already-idle. A bare launch takes 3.9s
  to render a composer at all.
- **Gate on cursor-in-composer**, over two consecutive readings. Residual: a
  cursor on any row *below* the glyph counts (which is what makes it work on a
  scrolled composer), so a bare shell whose prompt or scrollback carries a `❯`
  reads as a live composer and opens the gate at the shell.
- **Input loss is a discrete band, not a prefix** (measured 2026-09-21).
  Typing across a `claude --resume` boot: keystrokes from ~0.04s to ~1.7s
  arrive, a ~1–1.5s band at 1.7–3.3s vanishes, and everything after it
  arrives — `aoeuidhtnsaoe` came back as `aoeunsaoe`, intact head, intact
  tail, hole in the middle. The band closes ~2s *before* the alternate-screen
  switch (`#{alternate_on}` flips 0→1 at 4–6s), so it is over before anything
  the pane shows changes; what bounds it is unidentified.
- **From the cursor onward, input is buffered rather than lost.** The composer
  paints nothing for another 3–9s, load-dependent, then shows every keystroke
  sent meanwhile, in order (20 of 20 probe markers). Three consequences: a
  landing check must outlast that paint, a delay *before* sending buys
  nothing, and an empty composer seconds after a send is not evidence of a
  dropped keystroke.
- **A real `/exit` leaves no `❯` behind** — Claude Code runs on the alternate
  screen, so its frame goes with it and the restored screen is the shell's
  (CC 2.1.278). A `❯` seen under `--resume` comes from replayed content once
  the TUI is up, not from the exited session outliving it.
