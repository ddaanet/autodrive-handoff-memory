---
name: feedback_update_design_doc
description: Always update DESIGN.md after any design change to the handoff plugin
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 494dfa09-70b5-4ef8-bc70-603573015c26
---

After making any design change to the handoff plugin (skill behavior,
template, hook contract, content rules), update `DESIGN.md` in the same
pass — don't leave it as an optional follow-up.

**Why:** DESIGN.md is the living decision record; a change that lands in
code/skill but not in the doc makes the doc lie, and the next reader
trusts the doc. See [[feedback-design-doc-writing]] — strata are
dated and additive, so record the new decision as its own dated
subsection rather than rewriting old rationale.

**How to apply:** treat the DESIGN.md edit as part of the change, not a
proposal to offer afterward. Add a dated subsection capturing what
changed and why.
