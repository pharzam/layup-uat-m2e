# AGENTS.md

Agent context for **`tally`**, the Go command that counts the lines, words and
characters of text files, and for the engineering discipline its work follows.
Read this before you change anything in this repository.

This file is a startup index and summary of how we work here. Where it disagrees
with a document it points to or summarises, **the document wins** — and the
disagreement is a defect to fix in the same change
([R10](docs/issue-workflow.md#r10--sync-with-governance)).

## What this repository is

The repository `pharzam/layup-uat-m2e` holds the product `tally`: a command
`tally [-l] [-w] [-c] [file...]`, written in Go with the standard library only and
built with the Go toolchain, with a package `count` that counts the lines, the words
and the characters of a reader, apart from the package `main` of the command. The
[problem statement](docs/facts/problem-statement-brief.md) and the answers of
[F-0001](docs/facts/F-0001-setup-answers.md) say what it must do. The
repository started as a copy of the Armature baseline, pinned in
[`docs/setup/armature.pin`](docs/setup/armature.pin). Armature ships the "how we
work" — a quality gate, guardrails, decision records, a glossary, a
facts-and-requirements convention, a test section and a task backlog — and this
repository puts it to work on `tally`.

Two things catch agents out: each `‹…›` marker that is left is a value no one has
supplied yet, and it is deliberate; and the root [`tests/`](tests/) directory holds
the end-to-end and acceptance tests of the command, while the unit tests of the
package `count` sit next to its code, as Go expects.

## How these instructions rank

A platform or operator instruction of higher priority stays higher priority than
this file. Within its scope, this file governs work in this repository. The
documents under [`docs/`](docs/) are authoritative for their own subject, and this
file only summarises them. A nested instruction file, added lower in the tree, may
add a local constraint and may **never** weaken the quality gate. A conflict
between any two of them stops work until a decision note or an
[ADR](docs/adr/) resolves it.

## Start here

Read [`docs/engineering-discipline.md`](docs/engineering-discipline.md) for how we
work, then [`docs/issue-workflow.md`](docs/issue-workflow.md) for the ticket rules
that gate assumes.
[`docs/onboarding-for-engineers.md`](docs/onboarding-for-engineers.md) is the
human first door. Which document owns which class of rule is the table under
Sources of truth, written once.

## The product, in brief

- `-l`, `-w` and `-c` select the lines, the words and the characters; they combine,
  and the counts always come in that order. With no flag, all three.
- A row is the selected counts, then the file name, each separated by one space;
  the row for standard input has no name, and `-` is a file named `-`. With two
  files or more, the last row is the total of the files counted without an error,
  then `total`.
- A file that cannot be read, a directory, or a file that is not valid UTF-8 gives a
  message on standard error and no row, and the exit code is 1; the other files
  are still counted. Wrong usage gives the exit code 2; otherwise the exit code is 0.
- The targets: 10 MiB in at most 2 seconds, 1 GiB with at most 64 MiB of peak heap,
  and at least 90 percent statement coverage of the package `count`.

The vocabulary is in [`docs/glossary.md`](docs/glossary.md), and the traps of
counting text in [`docs/guardrails.md`](docs/guardrails.md) §2.

## The quality gate


Every substantive task passes **eight** ordered steps, in this order. Before step
1, an issue is open, and the work is sliced into an ordered, Definition-of-Done
covering, test-first plan that is reviewed once and recorded on the issue. That
plan review is architecture and scope, never implementation approval.

1. **Isolate.** Work in a per-task git worktree branched off `origin/main`, never in the operator's own checkout.
2. **Honor the guardrails.** Before you write code, read the acceptance criteria, [`docs/guardrails.md`](docs/guardrails.md), and the [ADRs](docs/adr/) the ticket references.
3. **Test first.** Write the failing test, watch it fail for the right reason, then write the code.
4. **Make long tasks visible.** Anything that can run over ten seconds shows which step runs and that it lives.
5. **Review until findings decay.** Freeze the head, then run independent blind rounds on it, a different lens each round. A fix re-freezes; at most two fix-and-review cycles follow the first freeze, and the last round ends `nothing material in scope` or `not mergeable, findings recorded` — see [Reviewing until findings decay](docs/engineering-discipline.md#reviewing-until-findings-decay). A defect the change *revealed*, off the path its Definition of Done names, opens an issue instead of entering the branch. A reviewer is a person or a fresh agent session — the requirement is [independence](docs/engineering-discipline.md#who-may-review), not reviewer type — and summarised text gets a clause-by-clause semantic pass.
6. **Be honest, keep evidence.** Report a failure as a failure, and review the producing code before a costly action.
7. **Keep the documentation current.** Every document the change leaves stale is fixed in the same pull request — and any lesson the task taught that the next reader could hit is written back into `guardrails.md` §2 (Known pitfalls).
8. **Close out in the same PR.** Tick the boxes, write the verdict, record the task line in the completed log, and — for a task started under ADR-0007 — write the task's resource record (model, effort, tokens and elapsed time per gate part, recorded not budgeted).

**Model tiers.** Where a step needs a model, a **reasoning tier** does the deciding
and judging — the plan and its review, solution selection, the review rounds, the
verdict — and an **execution tier** does tactical execution and coding once the plan
is fixed. Routing applies only after
[Determinism](docs/engineering-discipline.md#solution-selection) warrants a model at
all, a deterministic check still outranks any tier, and reviewer
[independence](docs/engineering-discipline.md#who-may-review) wins where it meets
routing. The rule and its tier-to-step map live in
[Model tiers](docs/engineering-discipline.md#model-tiers), recorded in
[ADR-0005](docs/adr/0005-route-work-by-model-tier.md).

## The issue rules

The workflow defines **thirteen** numbered rules, R1–R13; cite one by number in a
review or a commit. They are defined once, in
[`docs/issue-workflow.md`](docs/issue-workflow.md) — this file points there rather
than restating them, so there is a single source to keep in step. Which rule a
mechanism backs today, and which is `(written rule)` with none, is that document's
own [enforcement table](docs/issue-workflow.md#what-is-enforced-where), never this
file.

## Checks you can run

These read only text, so they need no toolchain. Install the hooks once per clone
by running `sh .githooks/install.sh` (it pins `core.hooksPath` to the relative
`.githooks`); the first four then run before every
commit, and in CI; `nested-checkout-check.sh` needs `git`, so it runs in CI only;
and `git diff --check` you run yourself. Keep
[`.gitattributes`](.gitattributes): it holds these scripts at line-feed endings,
without which none of them runs on a Windows checkout.

```
sh docs/adr/adr-lint.sh
sh docs/prd/prd-lint.sh
sh docs/links/link-lint.sh
sh docs/tests/run-discipline-tests.sh
sh docs/tests/nested-checkout-check.sh
git diff --check
```

[`docs/ci/pr-link-lint.sh`](docs/ci/pr-link-lint.sh) and
[`docs/ci/review-record-lint.sh`](docs/ci/review-record-lint.sh) read forge
artifacts, so they run in CI only and have no local run. The product checks of
`tally` use the Go toolchain: `go vet ./...`, `gofmt -l .` and `go test ./...`, and
`go test -cover ./count` for the coverage target. Never invent another build, lint
or test command; the targets and how each is measured are in
[`docs/guardrails.md`](docs/guardrails.md).

## Branches, worktrees, commits, and pull requests

Work in a per-task git worktree under `‹worktree dir›/<task>`, branched off
`origin/main`, never in the operator's own checkout. Commit at each logical step,
with a subject that follows Conventional Commits — `<type>: <ID> <description>`
when it carries a task. Rebase onto the latest `origin/main` and land with a plain
merge; **never squash** — but a branch already under a frozen-head verdict merges
`origin/main` in instead, so the reviewed SHA survives. The pull-request body
links its issue with `Closes #N`,
or `Refs #N` when it does not close it.

## The task index

[`docs/tasks/backlog.md`](docs/tasks/backlog.md) holds one line per open task, and
[`docs/tasks/completed.md`](docs/tasks/completed.md) the dated log of finished
ones. Any detail belongs in that task's own file, never in either index. The same
pull request that lands the work records the line in the completed log, moving it
from the backlog where the task had one.

## Placeholders and open values

Every `‹…›` marker that is left is a value only the operators of this project can
supply — the evidence store, the task-ID scheme, the worktree directory. Never put a
guess in its place, never invent a command, path or number of this project, and
never delete a marker to make a check pass. Search for the opening angle quote,
U+2039, to find every one of them.

## Safety limits

Never commit secrets, and never expose sensitive data. Never rewrite published
history — no force-push, no amend of a landed commit. Never run a destructive,
costly or irreversible operation without explicit authorization, and review the
code that will do the work before it runs. The full rule is
[Safety limits](docs/engineering-discipline.md#safety-limits).

## Decisions and questions

Record the plan, the selected option, the rejected alternatives, the tradeoffs and
the evidence on the issue — that is where a fresh context picks the work up. Ask
another operator on the issue thread, with a severity and an expected response
time, never directly. An architecturally significant decision becomes an
[ADR](docs/adr/).

## Sources of truth

| Document | Authoritative for |
|----------|-------------------|
| [`docs/engineering-discipline.md`](docs/engineering-discipline.md) | The quality gate, solution selection, testing, reviews, commits and the safety limits. |
| [`docs/issue-workflow.md`](docs/issue-workflow.md) | The numbered rules themselves, and the honest table of what a mechanism backs today. |
| [`docs/guardrails.md`](docs/guardrails.md) | Known pitfalls, pre-registered pass and fail rules, and how a result is validated. |
| [`docs/glossary.md`](docs/glossary.md) | The shared vocabulary, and the rule that every abbreviation earns an entry. |
| [`docs/adr/`](docs/adr/) | Architecture decisions that constitute a project, with the context and the consequences of each one. The past governance decisions of the Armature baseline were archived under `docs/decisions/`, which this repository removed when it was set up. |
| [`docs/tests/`](docs/tests/) | The test levels, a pattern for each, and the Definition-of-Done coverage checklist. |
| [`docs/facts/`](docs/facts/) and [`docs/prd/`](docs/prd/) | Customer facts kept as evidence, and the requirements derived from them. |
| [`.githooks/`](.githooks/) and [`docs/ci/`](docs/ci/) | What the gate enforces locally, and what CI enforces as the authority. |
