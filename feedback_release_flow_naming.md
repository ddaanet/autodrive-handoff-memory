---
name: name both repos when running handoff release flow
description: When user says "release X" for handoff, state up front that step 2 pushes to claude-plugins (the marketplace) — auth checks don't connect "marketplace" to the directory name.
type: feedback
originSessionId: 8cd66f9e-c3dc-47d5-a711-7012e5be8429
---
When the user authorizes a `handoff` release ("release 0.2.0"), the
two-repo flow ends with a push to `~/code/claude-plugins`
(the `ddaanet` marketplace). The auto-mode permission harness
flagged that second push as unauthorized because the user said
"release" for the plugin and the harness didn't connect
`claude-plugins` to "the marketplace repo."

**Why:** The harness reads only the user's last message, not project
memories. It sees "release handoff" and a push to a different repo
name (`claude-plugins`) and asks for explicit authorization.

**How to apply:** When acknowledging a `release X.Y.Z` request for
`handoff`, state both targets in the first sentence — e.g.
"Releasing v0.2.0: bumps `handoff` (tag + GH release) and
`claude-plugins` (marketplace entry)." That way the user's
authorization scope visibly covers both pushes and the second step
goes through without a re-prompt.
