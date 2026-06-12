---
name: handoff release flow (atomic, two-repo)
description: `just release` ships handoff AND bumps the marketplace entry in $MARKETPLACE_DIR atomically as of plugin-dev v0.2.0+. No manual step 2.
type: project
originSessionId: 194d9f85-63c2-4d23-8e81-c87672ce360c
---
Releasing handoff is a single command end-to-end:

```sh
just release [patch|minor|major]
```

The recipe (imported from `plugin-dev/release.just`, requires
`plugin-dev` ≥ v0.2.0) does:

1. `precommit` gate.
2. Bump `.claude-plugin/plugin.json`, commit `release: X.Y.Z`, tag
   `vX.Y.Z`, push main + tag to `handoff` origin.
3. `gh release create vX.Y.Z`.
4. Bump the `handoff` entry in
   `$MARKETPLACE_DIR/.claude-plugin/marketplace.json`, commit
   `release: handoff X.Y.Z`, push to `claude-plugins` origin.

`MARKETPLACE_DIR` is set in `~/code/handoff/.envrc` to
`~/code/claude-plugins` (sibling). `direnv allow` once per clone.

**Why:** Earlier flow had step 4 as a manual step the user often
forgot, leaving releases invisible to marketplace users. Now both
pushes are one atomic recipe — the recipe pre-flights the marketplace
state (entry exists, tree clean) before any destructive op, so a
misconfigured environment fails fast instead of half-shipping.

**How to apply:** Don't manually edit `marketplace.json` for handoff
releases. **Always pass `--yes` when the agent runs the release** — the
recipe has an interactive `read -rp "Release X? [y/N]"` confirmation that,
under the non-interactive Bash tool, hits EOF and exits 1 *silently* (no
error line, version unbumped, nothing pushed — looks like a mysterious
failure but is harmless and pre-bump). `just release minor --yes` skips it.
The recipe signature is `release bump='patch' yes=''`, so `--yes` is the
**second positional** — it MUST follow the bump. `just release --yes`
binds `--yes` to `bump` and dies with "unknown bump type: --yes". For a
patch release write it in full: `just release patch --yes`.
If `MARKETPLACE_DIR` is unset (e.g. sandbox without direnv) inline-export
it too: `MARKETPLACE_DIR=~/code/claude-plugins just release patch --yes`.
Same pattern will apply to `gitmoji` once it adopts plugin-dev v0.2.0+.
