---
name: directory-portal-validation
description: "`claude.ai/directory/manage` Validate, no CLI or API, `claude plugin validate --profile directory` passes what it refuses; drive it over CDP (`tab.js`), test on a pushed ref; `UNPINNED_NPX` \"Unpinned npx launcher\" with no npx, `COMMAND_PATH_COMPUTED` at `.`, `COMMAND_NAMES_MOD_FILE`, `python3 -c` or nested `${x%\"${y}\"}` in a hook script; \"accepted only from Verified organizations\" `LOCAL_CODE_*` blocks (command hooks, mods, `bin/`, scripts), no way to apply"
metadata:
  node_type: memory
  type: reference
  originSessionId: f16eeb4b-4407-42c5-b201-01b780af540f
  modified: 2026-11-06T15:58:17.932Z
---

The directory portal's Validate (claude.ai/directory/manage/new/plugin) has no CLI or API. `claude plugin validate --profile directory --strict` passed a tree the portal refused, and the Claude binary holds none of the rule codes. Observed 2026-11-05 with CC 2.1.318.

**Driving it.** The shared Chromium is on CDP port 9222, reached unsandboxed through `~/code/general/tools/tab.js`. `tab.js open` adopts any tab on the same host, so it can take over a tab my human partner has open: create a tab with `PUT /json/new?<url>` and register that target id instead. Validate takes seconds and saves nothing server-side; the portal locks the icon only at Save or Submit. The repo field takes `owner/repo@ref`, tags included, so old dist tags can be validated with no push. A fix can only be tested on a pushed ref.

**Bisecting.** Stub hook scripts in throwaway commits of the dist tree, force-pushed to one scratch branch, restoring one script at a time. Pushing runs the gitlore pre-push hook each time, which can take tier commits into `memory`.

**Shapes it refused in a hook script**, plugin at the repo root, where the docs say such checks only hold:
- A nested `${name%"${name##*[![:space:]]}"}` gave `UNPINNED_NPX` ("Unpinned npx launcher"), with no npx anywhere.
- An inline multi-line `python3 -c` in `write-guard.sh` gave `COMMAND_PATH_COMPUTED` at `.`, but only with `report-drive-failure.sh` also real. Unexplained: `load-handoff.sh` keeps an inline `python3 -c` and passes.
- `msg+="."` listed `.` as an unfollowed file.
- `x="${x:+$x$'\n\n'}$y"` in `load-compact.sh` gave `UNPINNED_NPX` too (2026-11-06); an `if [[ -n … ]]` plus `+=` passed. Both `UNPINNED_NPX` cases are a nested `${…}` expansion, so that shape is the one to suspect; the rule behind it is not known.
- A word naming `hooks/handoff.ts`, even in a comment, gave the `COMMAND_NAMES_MOD_FILE` hold.

The credential hold (`MCP_FORWARDS_CREDENTIAL_ENV`) is a hold, not a block, and moves rather than clears: rewording the lines it named in `bin/handoff-approved` shifted it to `_drive_lock.py` (an env var named at run time) and `_guard_paths.py`.

The rules tightened after 2026-10-15, so a tree that passed before may now be refused.

**Blocked since 2026-11-05 for non-Verified organizations.** Submitting refused 0.21.2 on four blocking rules: `LOCAL_CODE_HOOK_COMMAND`, `LOCAL_CODE_MOD`, `LOCAL_CODE_BIN` and `LOCAL_CODE_FILE_TYPE` (any file but Markdown, text, JSON, YAML or an image). Validate before submitting showed none of them. Anthropic designates Verified organizations and there is no way to apply (the portal says so; anthropics/claude-plugins-official#7024 asks how). This plugin cannot drop those parts, so it stays on the `claude-plugins` marketplace and the holds above are moot.
