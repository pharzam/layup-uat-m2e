# Onboarding for engineers

> **Who this is for.** Software engineers joining the work on `tally` who do
> **not** necessarily know how text is counted. You know how to build software.
> This document teaches you the terms of the project and the facts that shape
> every decision, then hands you off to the deeper documents.
>
> **Read this first.** Every other document is written for someone already fluent
> in these terms. This one gets you to that point. Budget half an hour.
>
> **Keep it in step.** This is the first document a new engineer reads, so a stale
> number here is worse than a stale number anywhere else (see the
> [plain-language-summaries](engineering-discipline.md#plain-language-summaries)
> rule). The numbers below come from the
> [problem statement](facts/problem-statement-brief.md) and from
> [F-0001](facts/F-0001-setup-answers.md); when one of them changes, this file
> changes in the same pull request.

## 1. Problem statement

A person who works in a terminal wants to know how long a text file is. `tally` is
a small command that reads the files the person names and prints the number of
lines, words and characters of each, and a total:

```
tally [-l] [-w] [-c] [file...]
```

The flag `-l` selects the lines, `-w` the words and `-c` the characters; with no
flag, `tally` prints all three. With no file name, it reads standard input. It is
written in Go, with the standard library only, and built with the Go toolchain.

It does not count bytes, paragraphs or sentences, it does not read archives or
directories, and it has no graphical interface, no network service and no plug-in
system.

The work succeeds when `tally` prints the counts of each named file and a total as
the behaviour rules say; when a file that cannot be read or is not valid UTF-8
gives a message and the exit code 1 while the other files are still counted; when
a file of 10 MiB is counted in at most 2 seconds and a file of 1 GiB with at most
64 MiB of peak heap; and when the package `count` has at least 90 percent statement
coverage (F-0001#15). Nothing is built yet.

## 2. Crash course: counting text

The terms build on each other, in this order. The same terms are in
[glossary.md](glossary.md), as definitions.

### 2.1 Character: one code point

Text is stored as bytes. In UTF-8, one Unicode **code point** takes one to four
bytes. `tally` counts code points, not bytes, and calls each one a **character**:

```
h   é      l   l   o
68  C3 A9  6C  6C  6F     six bytes, five characters
```

A combining accent is a code point of its own, so `e` followed by a combining acute
accent is two characters, though a reader sees one letter. A carriage return is a
character like any other (F-0001#2).

A file that is not valid UTF-8 is not text for `tally`: the file gets a message on
standard error, no row, and the exit code 1 (F-0001#6, F-0001#7).

### 2.2 Line: a run that ends with a line feed

A **line** is a run of characters that ends with a line feed. The last line counts
even when the file does not end with a line feed. A line feed alone ends an empty
line, which counts as one line. An empty file has zero lines (F-0001#2).

```
text        lines
""          0
"a"         1      (no final line feed, still a line)
"a\n"       1
"a\n\nb"    3      (the empty line counts)
"a\r\n"     1      (the carriage return is part of the line, and a character)
```

### 2.3 Word: a run that is not white space

A **word** is a run of characters that are not white space. White space is every
character of the Unicode property `White_Space` — not only the space, the tab and
the line feed, but also the no-break space, the ideographic space and others. So
`x=1;y` is one word, and `a`, a no-break space and `b` are two.

### 2.4 Row, total and exit code

Each file gives one **row**: the counts that the flags select, then the file name,
each separated from the next by one space. The counts always come in the order
lines, words, characters, whatever the order of the flags (F-0001#3). The row for
standard input has no name, and the name `-` is a file named `-`, not standard
input (F-0001#4).

```
$ tally notes.txt todo.txt
3 7 42 notes.txt
2 4 19 todo.txt
5 11 61 total
```

With two files or more, the last row is the **total**: the sum of the selected
counts of the files counted without an error, then `total` (F-0001#5).

The **exit code** is 0 when no error occurs, 1 when a file cannot be read, is a
directory, or is not valid UTF-8, and 2 for an unknown flag or wrong usage, which
prints a usage message and counts nothing (F-0001#7, F-0001#8).

## 3. What the system does

The command has two parts, in one Go module (F-0001#11):

```
package main                         package count
  read the flags        ----------->   count the lines, words and
  open each file                       characters of an io.Reader,
  print the rows,       <-----------   in one pass over the bytes
  the total, the errors
  and the exit code
```

The package `main` reads the flags, opens each file in turn, and hands its reader
to the package `count`. The package `count` reads the text in blocks, decodes the
UTF-8, and keeps three counters and one state: whether it is inside a word. It
reports an error when the text is not valid UTF-8. The package `main` prints a row
for each file that counted, a message on standard error for each that did not, the
total when there are two files or more, and ends with the exit code.

## 4. Why it is hard

Counting looks trivial, and the hard part is the three targets:

- **Time:** a file of 10 MiB in at most 2 seconds, measured as the elapsed time of
  the whole process, on a computer with 4 cores and a solid-state disk, with the
  file already in the cache (F-0001#9). This rules out a slow decoder, but not much
  else.
- **Memory:** a file of 1 GiB with at most 64 MiB of peak heap, sampled every 10
  milliseconds during the count (F-0001#10). The file is sixteen times larger than
  the allowed heap, so the obvious thing — read the whole file, then count it —
  does not work. The counter must stream the file in blocks, and a block boundary
  may fall inside a code point of several bytes or inside a word.
- **Coverage:** at least 90 percent of the statements of the package `count`, as
  `go test -cover` reports it (F-0001#11).

And the definitions are not those of `wc`: `White_Space` is wider than ASCII white
space, a character is a code point and not a byte, and a carriage return is a
character. A test that compares with `wc` proves nothing.

## 5. How this project works

### The one cultural thing to understand

The facts decide, not habit. Every behaviour of `tally` comes from a rule of the
problem statement or an answer of F-0001, and every one of them has a test written
before the code. When the facts do not say what to do, you ask on the issue; you do
not pick what `wc` does.

Consequences you will meet immediately, and which are not negotiable:

- Every substantive task passes the eight steps of the
  [quality gate](engineering-discipline.md#working-a-task-under-the-quality-gate),
  test first.
- The traps of counting text and the measurement of each target are in
  [`guardrails.md`](guardrails.md); read them before you write code, and do not
  move a target number to make a run pass.
- An open issue comes before any change ([`issue-workflow.md`](issue-workflow.md)).

### Repo layout

| Path | What |
|------|------|
| [`tests/`](../tests/) | The end-to-end and acceptance tests of the command. The unit tests of the package `count` sit next to its code. |
| [`AGENTS.md`](../AGENTS.md) | The agent entry point — the gate in brief and a pointer to the R1–R13 rules, in one short file. [`CLAUDE.md`](../CLAUDE.md) imports it for Claude Code. |
| [`engineering-discipline.md`](engineering-discipline.md) | **How we work**: the quality gate, solution selection, branches, worktrees, commits, tests, reviews, and ADRs. Read before your first commit. |
| [`glossary.md`](glossary.md) | The shared vocabulary. Skim it; come back constantly. |
| [`facts/`](facts/) | Facts collected from the customer, stored as-is as immutable evidence: the problem statement and the answers of F-0001. Derived requirements cite them by `F-NNNN` ID. |
| [`prd/`](prd/) | Product Requirements Documents, derived from the facts; each `REQ`/`NFR` cites an `F-NNNN` fact. |
| [`issue-workflow.md`](issue-workflow.md) | The issue-first rules (R1–R13): the ticket policy the gate assumes. |
| [`tasks/backlog.md`](tasks/backlog.md) | What to work on next. |

### What to read next, in order

0. [`AGENTS.md`](../AGENTS.md) — the one-page summary, if you are a coding agent
   (or a human who wants the shape before the detail).
1. [`engineering-discipline.md`](engineering-discipline.md) — how we work.
2. [`issue-workflow.md`](issue-workflow.md) — the issue-first rules the gate assumes.
3. [`glossary.md`](glossary.md) — skim, then reference.
4. [`guardrails.md`](guardrails.md) — the pitfalls and the frozen numbers.
5. [`tasks/backlog.md`](tasks/backlog.md) — what needs doing.

### Where the project stands

The repository is set up: the discipline is in place, the facts are recorded in
[`facts/`](facts/), and the targets are written down in
[`guardrails.md`](guardrails.md). No code of `tally` is written yet, and no
requirement document exists yet. The next step is to derive the requirements from
the facts into a PRD, and then to build the package `count` and the command, test
first.
