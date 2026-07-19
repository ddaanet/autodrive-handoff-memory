---
name: reference-jsonl-slash-command-shape
description: "How slash commands appear in Claude Code session JSONL — a <command-name> wrapper string, not literal prompt text or a Skill tool_use"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 6afc4947-de16-4c6c-9503-de03977fa611
---

In Claude Code session JSONL, a slash command like `/handoff:handoff`
is recorded as a `type:"user"` entry whose `message.content` is a
**string** containing wrapper tags, e.g.
`<command-name>/handoff:handoff</command-name>\n<command-message>handoff:handoff</command-message>\n<command-args>…</command-args>`
(leading slash present in `<command-name>`). It is NOT stored as the
literal text `/handoff:handoff`, and the slash path does NOT emit a
`Skill` tool_use.

By contrast, agent-invoked skills appear as an assistant `tool_use`
block: `{"type":"tool_use","name":"Skill","input":{"skill":"handoff:handoff"}}`.

**Skill bodies are injected as `isMeta:true` user entries** — on BOTH
paths: the `Skill` tool (the body entry also carries `sourceToolUseID`)
and the `/slash-command` path (no `sourceToolUseID`). The body is a
`content:[{type:text,…}]` block. Crucially it starts with the skill's
OWN first line (e.g. `# Update Config Skill`), not always
`"Base directory for this skill:"` — that preamble is only on some
(superpowers-style) skills, so a text-prefix filter misses native skill
bodies. A native body can be 100+ KB. `isMeta:true` is the robust
structural flag for ALL harness-injected non-conversation content (skill
bodies, `<local-command-caveat>`, `## Context`/`## Context Usage`
injections); genuine user prompts never carry it. (Historically the
removed `extract.py` dropped `isMeta`+`isSidechain` to keep skill content
out of the handoff — verified across 20+ real transcripts 2026-05-23.)

The handoff frame no longer scrapes the transcript for content, so `isMeta`
filtering is no longer used anywhere in the plugin. The one live scraper is
`handoff_activated()` in `scripts/_lib.sh`, an activation *detector* (not a
content extractor): it skips `isSidechain` entries and returns true on
either the agent path (a `Skill` tool_use with `input.skill` in
`handoff`/`handoff:handoff`) or the user path (a `type:"user"` entry whose
string content contains `<command-name>/handoff:handoff</command-name>`). It
does not inspect `isMeta`.

Verified against real transcripts 2026-05-23; `handoff_activated()` state
re-confirmed against live `_lib.sh` 2026-07-17. Relevant whenever detecting
skill activation or command invocation from a transcript. Sibling of
[[project_jsonl_sidechain_segregation]].
