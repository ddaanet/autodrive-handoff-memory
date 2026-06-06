---
name: project-hook-driven-skill-directives
description: "Design pattern for hook-driven turn-by-turn skill directives — PTU magic file for zero-turn injection, state machine in script for cross-session"
metadata: 
  node_type: memory
  type: project
  originSessionId: 209332e5-d743-407e-8d89-25879665a0b2
---

User is designing a system for turn-by-turn discovery of skill directives: a multi-step workflow injects one step at a time rather than front-loading all instructions.

**Architecture:**
- Agent completes a step → writes a magic state file → PTU hook fires within the same turn → injects next directive via `additionalContext` (zero extra turns)
- Same-session continuation: no hook machinery needed — agent resumes naturally with "continue"
- Cross-session continuation: handoff carries "activate skill X" → skill runs a script → script reads state file → emits correct directive for current step

**Key decisions:**
- PTU hook on a magic file write = zero-turn step injection (Stop hook still costs a turn)
- State machine logic lives in the script (cross-session path); hook is mechanical
- Explicit agent signaling required when completion requires judgment (e.g., TDD red phase — agent must confirm the failure is the intended one before signaling done)

**Why:** Progressive directive disclosure keeps agent context lean; hooks handle the happy-path step transitions without an extra round-trip.

**How to apply:** When this project comes up, the PTU magic-file pattern is the happy-path mechanism; script-in-skill is the cross-session fallback; explicit signal is the escape hatch for ambiguous completion.
