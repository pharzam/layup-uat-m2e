# T-ut9g — the requirements of tally

Issue #3, kind specification. In plain terms: this task writes the requirements
document of `tally` and its specification, so that each requirement points to
the exact words of the customer. It writes no Go code and no test of `tally`.

## Scope

- Requirements: REQ-001 to REQ-012 and NFR-013 to NFR-020, numbered as the draft
  rows R-001 to R-020; the draft rows R-021 to R-027 are success metrics, not
  requirements.
- Outputs: [PRD-0001](../prd/PRD-0001-tally.md), with the index row in
  [`docs/prd/README.md`](../prd/README.md), and the
  [specification of tally](../spec/tally.md).
- Facts: F-0003#n is fact n of the
  [problem statement](../facts/problem-statement-brief.md), Q-NNN an answer of
  the operators, and every row cites F-0002 too. The record F-0003 is rendered
  under `docs/facts/` at the first bet, not by this task.

## Decisions

- The MoSCoW and Phase cells of PRD-0001 are left empty: they are filled at the
  bet. Until then `sh docs/prd/prd-lint.sh` reports each row as having no
  MoSCoW value and no phase, and the rows that cite F-0003 resolve only once
  F-0003 exists.
- The draft criterion of NFR-015 cited R-009, the directory rule; the rejection
  of a file that is not valid UTF-8 is REQ-010, and the criterion cites REQ-010.
- One phase, `1`, the first release, ships every requirement.

## Definition of Done

- [x] §6 and §7 of PRD-0001 hold one row for each of the twenty IDs, each citing
  its facts and F-0002.
- [x] One acceptance row and one matrix row for each ID; one success row for each
  success-criterion row R-021 to R-027.
- [x] The specification holds one section for each ID.
- [ ] The bet fills MoSCoW and Phase, and F-0003 is rendered; `prd-lint` is green.
- [ ] Independent clause-by-clause semantic review recorded on the issue.

## Verdict

Open: the specification is written; the review and close-out follow.
