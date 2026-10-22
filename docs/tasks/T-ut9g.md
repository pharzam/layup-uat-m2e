# T-ut9g — the requirements of tally

In plain terms: this task writes one requirements document for `tally`, in which
each requirement points to the exact words of the customer, and one section of
explanation per requirement. It writes no Go code and no test of `tally`.

| Field | Value |
| ----- | ----- |
| Issue | #3 |
| Kind | specification |
| Requirements | REQ-001 to REQ-012, NFR-013 to NFR-020 |
| Base | `46bac1aaed4937070f99b9ef8aa27a18190089a1` |
| Budget maximum | 480 lines added plus removed, 8 files touched, against the base |
| Cycle cap | 2 fix-and-review cycles after the first freeze |
| Demo | A reader follows each assigned requirement of `tally` from [PRD-0001](../prd/PRD-0001-tally.md) to the words of the customer that it cites. |

## What it delivers

- [`docs/prd/PRD-0001-tally.md`](../prd/PRD-0001-tally.md): the thirteen sections of
  the [template](../prd/template.md). REQ-001 to REQ-012 in §6 and NFR-013 to NFR-020
  in §7 keep the numbers of the draft rows R-001 to R-020; each has its acceptance
  criterion. The draft's success criteria R-021 to R-027 are the seven metrics of §8.
  REQ-028 to REQ-030 are the `Won't` rows of the three non-goals, phase `—`. The §12
  matrix has one row per requirement, with the guardrail pitfall that applies and
  the task T-ut9g.
- [`docs/spec/tally.md`](../spec/tally.md): one section per requirement ID.
- The [PRD index](../prd/README.md) row of PRD-0001.

## How the facts are cited

A Facts cell cites `F-0003#n` for fact n of the recorded spans of the
[problem statement](../facts/problem-statement-brief.md), `Q-NNN` for an answer of
the setup, and always F-0002, the existing record. The F-0003 record itself is not
written by this step: the pipeline renders it under `docs/facts/` from the recorded
spans at the first bet. Its citations resolve from then on.

## The findings of the last attempt, and what this attempt does with each

| Finding | Disposition |
| ------- | ----------- |
| R12, MoSCoW and Phase empty on the assigned rows | Kept empty, as the step specification requires: the pipeline fills them at the bet. `prd-lint` reports these cells until then; this is expected, not hidden. |
| R12, F-0003 not captured, Q-NNN not mapped to F-0001 | The step specification forbids writing under `docs/facts/` and asks for `Q-NNN` citations; the pipeline renders F-0003 at the first bet. Every row cites its fact numbers of the spans, so the trace is complete once F-0003 exists. |
| R10, no `Won't` rows for the non-goals | Fixed: REQ-028 to REQ-030, `Won't`, phase `—`, each with its acceptance row, matrix row and spec section; NFR-014 to NFR-016 keep their meaning as constraints. |
| R12, Guardrail and Task cells empty in the matrix | Fixed: the matrix links the applicable pitfall of [`guardrails.md`](../guardrails.md) and names T-ut9g on every row; ADR and Test stay `—`, since none exists yet. |

## Corrections to the draft

- The draft criterion of NFR-015 cited "R-009" for the rejection of an invalid UTF-8
  file; R-009 is the directory rule. The PRD cites REQ-010 and records it in §11 and
  §13.

## Definition of Done

- [x] The PRD has every section of the template, with no `‹…›` marker left.
- [x] §6 and §7 hold one row per assigned ID, plus the three `Won't` rows, each with
      an acceptance row and a matrix row.
- [x] §8 holds the seven success metrics, each with its target, its measurement and
      its facts.
- [x] `docs/spec/` has a section per requirement ID.
- [ ] MoSCoW and Phase of the assigned rows set at the bet, and `prd-lint` green.
- [ ] F-0003 rendered at the first bet, and every cited fact number checked against it.
- [ ] An independent clause-by-clause semantic review recorded on the issue.

## Verdict

Not yet reviewed: the review rounds run on a frozen head after this step.
