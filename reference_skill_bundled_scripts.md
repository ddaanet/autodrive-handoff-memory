---
name: reference-skill-bundled-scripts
description: "how a skill body invokes its own bundled script: CLAUDE_PLUGIN_ROOT is UNSET in agent Bash; plugin bin/ is auto-added to PATH; invoke by bare name"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2374725e-5904-4dcf-b8f6-67e3f33aa929
---

How a SKILL.md body reliably invokes a script bundled in its own plugin —
**verified by test 2026-06-12** (CC 2.1.175), contradicting two claude-code-guide
passes that claimed it isn't possible:

- `CLAUDE_PLUGIN_ROOT` is **UNSET** in the agent's Bash tool environment (it is
  set only for hook processes). So `${CLAUDE_PLUGIN_ROOT}/scripts/foo.sh` from a
  skill body does NOT resolve. (`env` dump confirmed.)
- Claude Code **auto-adds every installed plugin's `<root>/bin` to PATH** — no
  `bin` key in `plugin.json` needed (the cache plugin bins are all on PATH).
  This holds from the cache path, so it works for end-users, not just a dev
  checkout.
- A `bin/` executable **resolves by bare name** from a fresh agent Bash call
  (tested: a throwaway `bin/handoff-probe-test` ran by name → printed output).
- The Skill tool also announces `Base directory for this skill: <abs-path>` at
  invocation; Anthropic's document-skills (xlsx/pptx/docx) lean on this with
  bare relative `python scripts/foo.py`. Less reliable across invocation paths
  than the `bin/`-on-PATH route.

**Canonical pattern:** put a uniquely-named thin shim in the plugin's `bin/`
(`exec bash "$(dirname "$0")/../scripts/real.sh" "$@"`), keep logic in
`scripts/`, and have the skill body invoke the shim by bare name. No hook
required for "skill body must run a bundled script." Use a hook only when no
agent action is needed on the result. See [[feedback-harness-over-agent]].
