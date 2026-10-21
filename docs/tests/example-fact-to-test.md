# Worked example — from a fact to a test

One concrete walk down the traceability line of this repository, so the abstract
chain in [`traceability-template.md`](traceability-template.md) has a complete
example beside it. It follows a single requirement of `tally` from the customer's
words all the way to the test that proves it, using only the conventions of the
baseline.

    fact (F-0001#2) → requirement (REQ-001) → guardrail → ADR → task (T-ab12) → test

This example needs no architecturally-significant decision, so the **ADR** link is
empty (`—`, as the [row at the end](#5-the-traceability-row-the-line-written-down)
shows); every other link is complete.

> **How this file is used.** This is an **illustrative** example: the fact is a
> real fact of `tally`, while the requirement ID, the task ID and the test name are
> shapes that the first PRD and the first task will settle. It names no test
> runner: the test's command stays the `‹unit test
> command›` placeholder throughout. When the real requirement and test exist, this
> page follows them.

## In plain terms

> Nothing in this project gets tested by accident. Every test traces back through
> a written requirement to something the customer actually said. This page shows
> that trace once, in full, so the pattern is concrete: a customer sentence
> becomes a requirement, the requirement inherits a known pitfall, and a test
> proves the pitfall is handled.

## 1. The fact (Layer 1 — the customer's words)

A [facts record](../facts/) stores the customer's words unchanged, numbered so a
requirement can cite one line. In [`facts/F-0001-setup-answers.md`](../facts/F-0001-setup-answers.md):

    2. `Q-001` A carriage return before a line feed is part of the line and is
       counted as a character, like any other code point. A line feed alone ends
       an empty line, which counts as one line. An empty file has zero lines,
       zero words and zero characters. — by idea-owner; …

This is `F-0001#2` — the second fact in facts document `F-0001`. It is immutable
evidence: it is not rewritten, only cited.

## 2. The requirement (Layer 2 — derived, and it cites the fact)

A [PRD](../prd/) requirement is written *from* that fact and cites it. In the
PRD's `§6` functional-requirements table:

    | REQ | Statement | MoSCoW | Phase | Facts |
    | --- | --------- | ------ | ----- | ----- |
    | REQ-001 | tally counts an empty line as one line and a carriage return as one character. | Must | 1 | F-0001#2 |

`REQ-001` is a stable ID, assigned once. Because it cites `F-0001#2`, a reader can
always trace the requirement back to the customer's exact words —
[`prd-lint.sh`](../prd/prd-lint.sh) checks that the cited fact resolves.

## 3. The guardrail (the known pitfall it inherits)

A counter that strips the carriage return, or that counts only the line feeds, is
a known trap of counting text, so it is in [`guardrails.md`](../guardrails.md) §2
as a known pitfall:

    - ❌ The last line without a line feed, and the carriage return.

The requirement now carries that guardrail: the test must prove that `a␍⏎⏎` has
two lines and four characters, not merely that `a⏎` has one line.

## 4. The test (strict TDD — red, then green)

The requirement is proved at the [unit level](test-levels.md) — the package
`count`, in isolation — following the [`template-unit.md`](template-unit.md)
pattern and the strict
[red-then-green](../engineering-discipline.md#requirements-traceability) order:

1. **Red.** Write the test first: count the text `a\r\n\n`, assert two lines, one
   word and four characters. Run `‹unit test command›`; watch it **fail** against
   code that still drops the carriage return — failing for the *right* reason.
2. **Green.** Count the carriage return as a character, run `‹unit test command›`
   again, watch it **pass**.
3. **Freeze.** Once a fresh context confirms it
   ([R9](../issue-workflow.md#r9--test-freeze-after-confirmation)), the test is
   frozen: not weakened later to make new code pass.

The test lives under `‹test directory›` and is tagged `unit` so it can run alone.

## 5. The traceability row (the line, written down)

Finally the test earns a row in the
[traceability table](traceability-template.md), closing the loop from fact to
test:

    | Test ID | Level | Covers | Fact | Guardrail | ADR | Task | Status |
    |---------|-------|--------|------|-----------|-----|------|--------|
    | crlf-and-empty-line | unit | REQ-001 | F-0001#2 | guardrails.md §2 | — | T-ab12 | frozen |

Read left to right, the row is the whole story: the customer said it (`F-0001#2`),
it became a requirement (`REQ-001`), it inherited a pitfall (`guardrails.md §2`), a
task delivered it (`T-ab12`), and a frozen unit test proves it. A requirement with
no such row is a requirement with no proof — which the
[DoD checklist](dod-checklist.md) treats as not done.
