# tests/

The home for the **product tests** of `tally` — the end-to-end and acceptance tests
of the command. It holds no test yet, because no code of `tally` is written yet.
The unit tests of the package `count` sit next to its code, as Go expects
(`count/count_test.go`); the tests that build the command and run it on files go
here. This directory is the default `‹test directory›`.

## In plain terms

> Put the tests that run the `tally` command on real files in this folder. The
> tests of the package `count` live beside its code. The rules for what to write
> and how live one folder over, in `docs/tests/`.

## Where the conventions live

This directory holds the tests; the **conventions** for writing them live in
[`docs/tests/`](../docs/tests/):

- [`docs/tests/test-levels.md`](../docs/tests/test-levels.md) — the test levels and
  the command placeholders.
- [`docs/tests/template-unit.md`](../docs/tests/template-unit.md),
  [`template-integration.md`](../docs/tests/template-integration.md),
  [`template-e2e.md`](../docs/tests/template-e2e.md) — a pattern to copy per level.
- [`docs/tests/traceability-template.md`](../docs/tests/traceability-template.md) —
  the row that ties each test back to the requirement it proves.

The repository's own [discipline tests](../docs/tests/test-levels.md) — the ADR, PRD,
link, PR-link and review-record linters — are not product tests and do
**not** live here; they stay beside the conventions they enforce, under
[`docs/`](../docs/).

> **How this directory is kept.** Add the end-to-end tests of `tally` here, and set
> the `‹test
> directory›` placeholder in [`docs/tests/test-levels.md`](../docs/tests/test-levels.md)
> and the [hook](../.githooks/pre-commit)/[CI](../docs/ci/) steps to point at it.
> The `.gitkeep` file only exists to keep this empty directory in git — delete it
> once the first real test is in.
