---
name: feedback_update_design_doc
description: Always update DESIGN.md after any design change to the handoff plugin
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 494dfa09-70b5-4ef8-bc70-603573015c26
  modified: 2026-07-25T09:40:26.862Z
---

After making any design change to the handoff plugin (skill behavior,
template, hook contract, content rules), update `DESIGN.md` in the same
pass — don't leave it as an optional follow-up.

**Why:** DESIGN.md is the living decision record; a change that lands in
code/skill but not in the doc makes the doc lie, and the next reader
trusts the doc. See [[feedback-design-doc-writing]] — strata are
dated and additive, so record the new decision as its own dated
subsection rather than rewriting old rationale.

Additive is only half the contract. When the new decision *reverses* an
earlier one, a new subsection alone leaves the old section reading as
current, and a reader who stops there is misled by rationale that was
true when written. The doc's own device is a `> **Superseded <date>**
(see *<section>*)` blockquote at the head of the invalidated section,
scoped to what actually changed — mechanism, a single exclusion, a count
— so the surviving reasoning stays readable.

**How to apply:** treat the DESIGN.md edit as part of the change, not a
proposal to offer afterward. Add a dated subsection capturing what
changed and why, and in the same pass mark every section the change
invalidates. Unmarked reversals accumulate silently: eight of them had
piled up by 2026-07-25, against six correctly marked.
