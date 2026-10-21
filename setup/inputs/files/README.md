# tally

*A small Go command that counts the lines, words and characters of text files.*

This is the repository `pharzam/layup-uat-m2e`. It holds the product `tally` and
the engineering discipline that its work follows: a quality gate, guardrails,
ADRs, a glossary, the customer facts, the requirements and a task backlog.

## What tally does

A person who works in a terminal wants to know how long a text file is. `tally`
reads the files that the person names and prints the number of lines, words and
characters of each, and a total:

```
tally [-l] [-w] [-c] [file...]
```

- `-l` prints the lines only, `-w` the words only, `-c` the characters only. The
  flags combine; the counts always come in the order lines, words, characters.
  With no flag, `tally` prints all three.
- With no file name, `tally` reads standard input, and its row has no name.
- A line ends with a line feed; a word is a run of characters that are not
  Unicode `White_Space`; a character is one Unicode code point of UTF-8 text.
- A file that cannot be read, or that is not valid UTF-8, gives a message on
  standard error, adds no row and nothing to the total, and makes the exit code 1.
  The other files are still counted. Wrong usage gives the exit code 2.

The quality targets: a file of 10 MiB in at most 2 seconds on a computer with 4
cores, a file of 1 GiB with at most 64 MiB of peak heap, and at least 90 percent
statement coverage of the package `count`.

`tally` is written in Go, with the standard library only, and built with the Go
toolchain.

## Start here

1. **[`docs/onboarding-for-engineers.md`](docs/onboarding-for-engineers.md)** — read this first
   (about 30 minutes). It states the [Problem statement](docs/onboarding-for-engineers.md#1-problem-statement)
   and teaches the vocabulary of `tally`.
2. **[`docs/engineering-discipline.md`](docs/engineering-discipline.md)** — how we work: the
   quality gate every substantive task passes, plus branches, tests, reviews, and
   ADRs. Read it before your first commit.
3. **[`AGENTS.md`](AGENTS.md)** — if you are a coding agent, start here instead. It
   summarises [`docs/engineering-discipline.md`](docs/engineering-discipline.md)
   and [`docs/issue-workflow.md`](docs/issue-workflow.md), and indexes the rest.
   The long documents stay authoritative.

## What's inside

| Piece | What it holds |
|-------|---------------|
| [`AGENTS.md`](AGENTS.md) | The agent entry point: the quality gate, the checks, a pointer to the R1–R13 rules, and which document is authoritative for each. |
| [`CLAUDE.md`](CLAUDE.md) | One line, `@AGENTS.md`, so Claude Code loads the same guide. No second copy to drift. |
| [`docs/agents/`](docs/agents/) | What the entry points are and what they may not become ([ADR-0004](docs/adr/0004-ship-agent-entry-points.md)). |
| [`docs/onboarding-for-engineers.md`](docs/onboarding-for-engineers.md) | The first door: the problem statement and a crash course in counting text. |
| [`docs/engineering-discipline.md`](docs/engineering-discipline.md) | The quality gate, the solution-selection standard, and every working practice. |
| [`docs/issue-workflow.md`](docs/issue-workflow.md) | The issue-first workflow (R1–R13): the ticket policy the gate assumes. |
| [`docs/glossary.md`](docs/glossary.md) | The shared vocabulary the other documents assume. |
| [`docs/guardrails.md`](docs/guardrails.md) | Known pitfalls, pre-registered pass and fail rules, and validation. |
| [`docs/adr/`](docs/adr/) | Architecture Decision Records — the *why* behind structural choices — plus [`adr-lint.sh`](docs/adr/adr-lint.sh), the discipline test that keeps them honest. |
| [`docs/facts/`](docs/facts/) | The customer facts kept as immutable evidence: the [problem statement](docs/facts/problem-statement-brief.md) and the [answers of the setup](docs/facts/F-0001-setup-answers.md). |
| [`docs/prd/`](docs/prd/) | Product Requirements Documents derived from the facts, plus [`prd-lint.sh`](docs/prd/prd-lint.sh), the discipline test that keeps them honest. |
| [`docs/tests/`](docs/tests/) | The testing conventions — the test levels, a pattern per level, the security, scaling and Definition-of-Done checklists, and test-to-requirement traceability — plus [`run-discipline-tests.sh`](docs/tests/run-discipline-tests.sh), which tests the linters of the baseline against fixtures. |
| [`tests/`](tests/) | The root directory for the end-to-end and acceptance tests of `tally`. |
| [`docs/tasks/`](docs/tasks/) | The task index — [`backlog.md`](docs/tasks/backlog.md) and [`completed.md`](docs/tasks/completed.md). |
| [`.githooks/`](.githooks/) | Git hooks that enforce the cheap gate locally — a commit-message check and a pre-commit runner. Install them with `sh .githooks/install.sh`. |
| [`.gitattributes`](.gitattributes) | Keeps the scripts and hooks at line-feed endings, without which none of them runs on a Windows checkout, and pins the fixtures whose Windows endings *are* the assertion. |
| [`docs/ci/`](docs/ci/) | The CI files (GitHub Actions and GitLab CI) that run the same gate on every pull request; the GitHub ones are in place under `.github/workflows/`. |
| [`docs/templates/`](docs/templates/) | Inert GitHub and GitLab issue and pull-request forms that embody the issue-first workflow. |

## Where this repository came from

This repository started as a copy of Armature, the engineering-discipline
baseline at `https://github.com/pharzam/armature`. The pin file
[`docs/setup/armature.pin`](docs/setup/armature.pin) records the source, the
commit, the tree, the method and the date of that copy, and
[ADR-0009](docs/adr/0009-pin-the-baseline.md) records the decision to pin it. The
copy keeps no link back to Armature: a later change of the baseline comes in only
by a change that names the new commit.

LAYUP, which set up this repository and delivers `tally`, keeps its own records of
the delivery on the branch `layup-records`, apart from `main`. The product and its
documents live on `main`.

## Turning on enforcement

Install the git hooks by running `sh .githooks/install.sh` (it pins
`core.hooksPath` to the relative `.githooks`). The
[ADR](docs/adr/adr-lint.sh) and [PRD](docs/prd/prd-lint.sh) linters and their
[fixture self-tests](docs/tests/run-discipline-tests.sh) need no toolchain; the
checks of `tally` itself need the Go toolchain. Search for the opening angle quote, U+2039, to find each value
that is still open.
