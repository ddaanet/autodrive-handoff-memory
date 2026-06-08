---
name: project-test-suite-framework-port
description: "handoff test suite ported to bats (hooks) + pytest (extract.py); conversions, fidelity + quality reviews, AND full integration DONE — green via just precommit (37 bats + 77 pytest)"
metadata: 
  node_type: memory
  type: project
  originSessionId: d88d17aa-b0e8-4172-ae43-273c924cb8cf
---

David is replacing handoff's hand-rolled bash test suite with real
frameworks. Binding constraint: **ergonomics** (the `run`/`$status` win
and killing the `set +e/rc=$?/set -e` dance), not hermeticity.

**State as of 2026-06-08 — DONE. Full integration executed; `just
precommit` green (36 bats + 81 pytest).** Originals `git rm`'d (staged);
new files untracked, not yet committed (David hasn't asked to commit).

Integration as built (diverged from the uv plan below per
[[feedback-uv-direnv-venv]] — David dropped `uv run`):
- `pyproject.toml`: virtual project (`[tool.uv] package=false`), dev dep
  `pytest>=8`, `[tool.pytest.ini_options] pythonpath=["scripts"]` +
  `testpaths=["tests"]`. `uv sync` → `.venv` + committed `uv.lock`.
- `.envrc`: exports `VIRTUAL_ENV=$PWD/.venv` + `PATH_add $VIRTUAL_ENV/bin`
  so direnv activation puts bare `pytest` on PATH. `.venv/`,
  `.pytest_cache/` gitignored.
- `conftest.py` deleted (pythonpath replaces the sys.path hack).
- `justfile`: precommit = jq lints + ast parse + `shellcheck -x
  scripts/*.sh tests/*.sh tests/*.bats` + `bats tests/*.bats` +
  `pytest`. `hook-test`/`extract-test` recipes updated.
- Cleanups applied: rename-test.bats `run env PATH=… bats`; all
  `mktemp -d` → `$BATS_TEST_TMPDIR` subdirs (teardowns dropped); pytest
  render unified to one `render_frame` (redirect_stdout) helper; `.index`
  → `assert_order`; dropped redundant bare-`>` + omission-marker render
  greps (covered by `TestFormatQuote`/`TestClampAnchorLines`) → 83→81.
- CLAUDE.md Testing section rewritten for bats/pytest/direnv.

Done (in the working tree, originals `git rm`'d, not yet committed):
- `tests/hook-test.sh` + `tests/rename-test.sh` → `tests/hook-test.bats`
  (30 `@test`) + `tests/rename-test.bats` (6). **bats-core only, no
  helper libs.** 36/36 green.
- `tests/extract-test.sh` → `tests/test_extract.py` (83 tests) +
  `tests/conftest.py` (prepends `scripts/` to `sys.path`). Green via
  `uv run --with pytest pytest`. extract.py imports safely (`__main__`
  guard); unit tests on pure functions + end-to-end renders preserved.
- Reviews: pytest faithful + idiomatic; bats faithful (yes-with-nits).
  The bats-quality "HIGH masked-assertion" finding was a **verified false
  alarm** — see [[reference-bats-errexit]], [[feedback-verify-review-findings]].
- Docs written: `tests/bats-best-practices.md`,
  `tests/pytest-best-practices.md` (each ends with a reviewer checklist).

**Quality review (2026-06-08, the previously-skipped half).** Two Sonnet
reviewers audited the integrated tests against the rubrics in
`tests/{bats,pytest}-best-practices.md`. No HIGH/correctness defects.
Applied: pytest — added `addopts=[--strict-markers,--strict-config,
--import-mode=importlib]`; added `main([])`→exit-2 usage contract test
(`TestCliContract`); restored render-level marker+ordering coverage as
`test_eight_line_anchor_truncated` via `assert_order` (the earlier N3
cleanup over-pruned it — a cross-cutting frame property the unit test
can't discharge, rubric §2); dropped subprocess `check=True` for an
explicit `returncode==0` assert. bats — split the dual-scenario
load-handoff test in two; guarded setup's `[ -f ]`. Declined (settled
style per [[reference-bats-errexit]]): the uniformly-unguarded
`[ ]`/`[[ ]]` assertions and `run`-on-success-cases.
`pytest-best-practices.md` was then rewritten to match the current repo
(no conftest, `render_frame`/`assert_order`, `uv sync`+direnv, no
`uv run`). Both rubrics were finally **independently fact-checked +
gap-filled** by per-doc agents against live sources and empirically
(pytest 9.0.3/Py3.14.3; bats 1.11.1/Bash 5.2). Two real doc bugs found
& fixed (orchestrator re-verified both): (1) pytest §8 showed
`pytest.raises(SystemExit)` for `main([])` — wrong: `main(argv)` RETURNS
2 (no raise), and `main([])` IndexErrors on `argv[0]`; use
`main(["extract.py"]) == 2`. (2) bats §1c's `! cmd || false` fallback is
broken — bats's preprocessor mangles a leading `!` into a literal `!`
command → status 127; use `run ! cmd` or `cmd && return 1 || true`.
Also: pytest `--import-mode=importlib` is NOT the pytest-9 default
(`prepend` still is); BW02 (not BW01) warns on `run` flags without
`bats_require_minimum_version 1.5.0`. Gap-fills added: pytest §11
(warnings-as-errors, dev-loop flags, monkeypatch-vs-mock); bats §8
(PATH-stub mocking — the pattern rename-test already uses — parameterized
tests, diagnostics flags).

The open uv-sandbox decision is **resolved**: dropped `uv run` entirely
(direnv-activated venv instead, [[feedback-uv-direnv-venv]]), so neither
`pytest` nor `just precommit` needs unsandboxed execution — only the
occasional `uv sync` does.

See [[reference-shellcheck-lints-bats]] (bats lints natively, killed the
main objection).
