# Plan of T-ut9g — the requirements of tally (issue #3, kind specification)

In plain terms: this task writes one requirements document for `tally`. In it, each requirement points to the exact words of the customer. It writes no Go code and no test of `tally`.

## 1. Goal, demo, scale

| Item | Value |
| ---- | ----- |
| Demo (R11) | A reader follows each of the twenty assigned requirements of `tally` from `docs/prd/PRD-0001-tally.md` to the unchanged words of the customer in F-0003 or F-0001. |
| Goal classes in the DoD (R11) | 1: "each assigned requirement traces to the exact customer words". D2 to D5 below are the parts of that one trace. The fact record F-0003 is a precondition of the trace, because `prd-lint` fails when a cited fact file is missing. Close-out (D7) is gate step 8 and is not a goal. If the reviewer counts more than one class, split F-0003 into a child issue that lands first. |
| Base SHA | `46bac1aaed4937070f99b9ef8aa27a18190089a1` (`origin/main` expected). If `origin/main` is at another commit when the worktree is made, record the new SHA on the issue before step 3, and measure the budget against it. |
| Budget maximum | 480 lines added plus removed on the whole branch diff, and 8 files touched, against the base SHA. This includes the close-out bookkeeping. |
| Cycle cap | 2 fix-and-review cycles after the first freeze (the repository maximum). |
| Size | small: about 340 changed lines in 8 files, all Markdown. |

The 8 files: `docs/facts/F-0003-problem-statement.md` (new), `docs/facts/README.md`, `docs/prd/PRD-0001-tally.md` (new), `docs/prd/README.md`, `docs/glossary.md`, `docs/guardrails.md`, `docs/tasks/T-ut9g.md` (new), `docs/tasks/completed.md`. `docs/tasks/backlog.md` has no line for T-ut9g, so it does not change. `docs/facts/problem-statement-brief.md` and `docs/setup/facts.sha256` do not change.

## 2. One reading of "Must, phase 1" (R13)

The twenty assigned rows — REQ-001 to REQ-012 in §6 and NFR-013 to NFR-020 in §7 — are Must, phase 1. Three new rows, REQ-028, REQ-029 and REQ-030 in §6, are Won't, phase `—`. The PRD has no other requirement row. The draft rows R-021 to R-027 are success metrics in §8, not requirements.

## 3. Definition of Done

| DoD | Item |
| --- | ---- |
| D1 | `docs/prd/PRD-0001-tally.md` has all 13 sections of `docs/prd/template.md`, with no `‹…›` marker left. |
| D2 | §6 and §7 hold exactly the 23 rows of section 2. Each row keeps the meaning of its draft row (draft R-nnn becomes REQ-nnn or NFR-nnn with the same number) and cites its facts by this map: draft cover `n` becomes `F-0003#n`; draft cover `Q-00k` becomes `F-0001#(k+1)` (Q-014 becomes `F-0001#15`). REQ-028, REQ-029 and REQ-030 cite `F-0003#7`, `F-0003#8` and `F-0003#9`. |
| D3 | `docs/facts/F-0003-problem-statement.md` holds the 23 numbered facts of the problem statement. The text of fact n, after its `n. ` prefix, is byte for byte the span of fact n that the setup recorded (`spans.tsv`, byte offsets into `docs/facts/problem-statement-brief.md`). The header links the original, which stays unchanged. |
| D4 | §8 holds R-021 to R-027, each with its target and its measurement: R-021 is a 10 MiB (10,485,760 bytes) ASCII file with lines of about 80 characters, read once to warm the cache, on 4 cores and a solid-state disk, at most 2 seconds of elapsed time of the whole process from start to exit (F-0003#18, F-0003#22, F-0001#9). R-022 is a 1 GiB (1,073,741,824 bytes) file with `HeapAlloc` from `runtime.ReadMemStats` sampled every 10 ms, and the largest sample at most 64 MiB (67,108,864 bytes) (F-0003#19, F-0003#23, F-0001#10). R-023 is `go test -cover` on the package `count` at least 90.0 percent, with `main` outside the target (F-0003#20, F-0001#11). R-024 to R-027 are the four clauses of F-0001#15. Each of these four is measured by the requirements it names: REQ-001 to REQ-006, REQ-007 to REQ-010, R-021 with R-022, and R-023. |
| D5 | The §12 matrix has one row for each of the 23 requirements: facts, guardrail, ADR, task and test (see step 6). |
| D6 | The documents that the change makes stale are current: the facts and PRD indexes, the glossary, and a lesson in `guardrails.md` §2. |
| D7 | Semantic agreement: an independent review record of a clause-by-clause pass (dod-checklist, "The one item a test cannot close"). |
| D8 | Close-out: the verdict, the resource record (ADR-0007), the completed-log line, and the ticked DoD checklist. |

## 4. Checks of this specification

These checks test this document only. They are not tests of `tally`. A specification task adds no Go test and no file under `tests/`, so the §12 Test column is `—` on every row until a build task writes the test. The project checks are those that AGENTS.md names. V2 to V5 are one-off read-only comparisons. Their output goes on the issue as evidence. They are not added to the repository and are not a new project command.

| Check | What it proves | Red before the documents exist (step 3) | Green after |
| ----- | -------------- | -------------------------------------- | ----------- |
| V1 `sh docs/prd/prd-lint.sh` | IDs unique, a fact file resolves for each row, MoSCoW value allowed, Won't gives phase `—`, matrix ID set equals row set | Run on a scratch copy outside the repository (`prd/` with `PRD-0001-tally.md` copied from the template, and `facts/`): it fails with "no requirement rows". A second scratch run with a row that cites `F-0003` fails with "unknown fact" while F-0003 is missing. | Step 6, on `docs/prd` |
| V2 ID set | The IDs of §6 and §7 are exactly REQ-001 to REQ-012, NFR-013 to NFR-020 and REQ-028 to REQ-030; §12 holds the same set; each Must row has phase 1, each Won't row phase `—` | Fails: the file does not exist | Step 6 |
| V3 verbatim facts | For each n from 1 to 23, the bytes of fact n in F-0003 equal the recorded span | Fails: F-0003 does not exist | Step 4 |
| V4 citation map | The Facts cell of each row equals the D2 map | Fails: the file does not exist | Step 5 |
| V5 metrics | §8 holds the seven IDs and the tokens `10,485,760`, `1,073,741,824`, `67,108,864`, `10 ms`, `HeapAlloc`, `90.0`, `start to exit`, `warm` | Fails: the file does not exist | Step 5 |
| V6 `sh docs/links/link-lint.sh`, `sh docs/adr/adr-lint.sh`, `sh docs/tests/run-discipline-tests.sh`, `git diff --check` | Links resolve, discipline self-tests stay green | Green at base; must stay green | Every commit |

V1 also confirms that the acceptance-criteria lists of step 5 (numbered lines, not table rows) are not read as duplicate requirement rows. If they are read as rows, the criteria go into §5 instead, and the template stays unchanged.

## 5. Ordered steps

1. Isolate (gate 1). Make a per-task worktree off `origin/main`, on the branch that the harness gives. The `‹worktree dir›` marker is an open gap; do not invent a path. Run `sh .githooks/install.sh`. Check the base SHA (section 1). Covers: none (precondition).
2. Guardrails (gate 2). Read `docs/guardrails.md` §1 and §2, ADR-0002, ADR-0008, `docs/prd/README.md`, `docs/prd/template.md`, `docs/facts/README.md` and `docs/tests/dod-checklist.md`. Covers: precondition of D1 to D5.
3. Verification slice, red (gate 3, R8). Write V1 to V5 as commands in the issue comment. Run them before any document exists, and record that each fails for the reason given in section 4. No commit: the hooks refuse a red `prd-lint`. Covers: D2 to D5.
4. Facts capture. Write `docs/facts/F-0003-problem-statement.md` from `docs/facts/template.md`. Header: source is the problem statement of `tally`; origin is `docs/facts/problem-statement-brief.md`, kept unchanged and linked; status is Raw. Body: facts 1 to 23 in span order. Note in "Notes on capture" that facts 14 and 15 share one source line, and that the headings are not facts. Add the F-0003 row to the facts index. V3 becomes green. Commit `docs: T-ut9g record the problem statement as F-0003`. Covers: D3, part of D6.
5. Requirement derivation. Write `docs/prd/PRD-0001-tally.md`: §1 header (Derives from F-0003, F-0001) and a plain-terms summary; §2 problem with cited facts; §3 goals, and the three non-goals of F-0003#7 to #9; §4 one persona, the terminal user (F-0003#2); §5 user stories with citations; §6 REQ-001 to REQ-012 (Must, 1) and REQ-028 to REQ-030 (Won't, `—`), each followed by its acceptance criterion from the draft; §7 NFR-013 to NFR-020 (Must, 1), with criteria; §8 metrics of D4; §9 phase 1 is the first release of `tally` with all Must rows, and phase `—` is never shipped; §10 risks, linked to the §2 pitfalls of the guardrails; §11 assumptions; §13 change log. The statements are short, active and present tense, and keep the meaning of the draft. NFR-014, NFR-015 and NFR-016 stay Must constraints. The Won't rows name the excluded capability: other counts (REQ-028); unpacking archives or walking directories (REQ-029); a graphical interface, network service or plug-in system (REQ-030). Correct the NFR-015 criterion: an invalid UTF-8 file is rejected "as REQ-010 states", not R-009 (R-009 is the directory rule). Record this correction in §11 and §13. V4 and V5 become green. Commit `docs: T-ut9g derive PRD-0001 for tally`. Covers: D1, D2, D4.
6. Traceability. Write §12 with 23 rows. Facts as in D2. Guardrail: REQ-004 flag order; REQ-005 the name `-`; REQ-006 to REQ-010 an error that stops the run; NFR-018 last line and carriage return; NFR-019 ASCII white space; NFR-020 byte count and block boundary; the other rows §1 pre-registered behaviour, or `—`. ADR `—` on each row: no ADR of `tally` exists. Task: T-ut9g, the only real task ID; no build-task ID is invented. Test `—`. V1 and V2 become green on `docs/prd`. Commit `docs: T-ut9g add the traceability matrix of PRD-0001`. Covers: D5.
7. Documentation (gate 7). In `docs/prd/README.md`, add the index row `PRD-0001 | tally | Accepted` and replace "No PRD is written yet.". In `docs/glossary.md`, add MiB, GiB and peak heap (`HeapAlloc`), the terms the PRD uses that have no entry. In `docs/guardrails.md` §2, add the lesson: a draft criterion can cite a requirement by the wrong number; check each cross-reference against the statement. V6 stays green. Commit `docs: T-ut9g bring the indexes and the glossary up to date`. Covers: D6.
8. Freeze and review (gate 5). Freeze the head. Round 1, semantic lens: a fresh reasoning-tier session does a clause-by-clause pass of each of the 23 rows, each criterion and each of the 7 metrics against F-0003 and F-0001. Round 2, conformance lens: template and README conventions, the R13 second reading, and the budget. A fix re-freezes, within the cycle cap. The last round ends `nothing material in scope` or `not mergeable, findings recorded`. A defect found off this path opens an issue and does not enter the branch. One such defect is known now: F-0002 has no row in the facts index. Covers: D7.
9. Close-out (gate 6 and 8). Write `docs/tasks/T-ut9g.md` with the DoD checklist ticked, `## Verdict` and the resource record (part, expected tier, model, effort, tokens, elapsed; `not reported` where the harness gives no figure). Add the line to `docs/tasks/completed.md`. The pull request body says `Closes #3`. Rebase onto `origin/main` and land with a plain merge, never squash. After the frozen-head verdict, merge `origin/main` in instead of a rebase. Covers: D8.

## 6. Selected approach and alternatives

| Option | Decision | Tradeoff |
| ------ | -------- | -------- |
| A. F-0003 holds the problem statement verbatim; one PRD-0001; new Won't rows REQ-028 to REQ-030 for the non-goals | Selected | Three IDs are added beyond the assigned set, because template §3 requires a Won't row for each non-goal. |
| B. Cite `problem-statement-brief.md` directly, without F-0003 | Rejected | It has no F-ID and no fact numbers, so `prd-lint` cannot resolve a citation and a reader cannot cite one fact. |
| C. Mark NFR-014, NFR-015 and NFR-016 Won't | Rejected | This changes the meaning of assigned IDs: they are constraints that the product must keep, not excluded capabilities. |
| D. Number the Won't rows REQ-021 to REQ-023 | Rejected | It breaks the one-to-one map from draft R-nnn to REQ-nnn or NFR-nnn, and it looks like the metrics R-021 to R-023. |
| E. Commit a verbatim checker or extend `prd-lint` | Rejected for this task | It is a new interface and a second goal. Candidate follow-up issue. |
| F. Split into two issues, facts then PRD | Rejected | The PRD cannot pass `prd-lint` without F-0003, and the two give one demo. Kept as the fallback of section 1. |

## 7. Risks

1. `prd-lint` can read the criteria lists as rows. V1 in step 3 detects this; the fallback is in section 4.
2. A fact that starts with `- `, `1. ` or `|` renders as nested Markdown in F-0003. This is accepted: byte fidelity outranks rendering, and V3 checks bytes.
3. R-024 to R-027 have no statement of their own in the draft. The PRD uses the F-0001#15 clauses as targets, and the review checks this.
4. `origin/main` can move. The base SHA is checked in step 1.

size: small
open questions: —
new interfaces: two Markdown documents and no code — docs/facts/F-0003-problem-statement.md (the 23 verbatim facts of the problem statement, cited as F-0003#n) and docs/prd/PRD-0001-tally.md (REQ-001..REQ-012 and NFR-013..NFR-020 Must phase 1, REQ-028..REQ-030 Won't phase —, metrics R-021..R-027, §12 matrix)
new dependencies: —
tests:
