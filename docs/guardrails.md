# Guardrails — the known pitfalls, and the numbers you must not move

This document is the target of gate step 2, "Honor the guardrails", in
[`engineering-discipline.md`](engineering-discipline.md). It holds the pitfalls a
task author must take into account **before** writing code, so the team does not
re-derive a known trap every time.

It is written for `tally`, the Go command that counts the lines, words and
characters of text files. It merges two kinds of guardrail that many projects keep
in separate files — **decision gates** (pre-registered pass/fail rules) and
**validation** (how you check you are not fooling yourself). Both are read before
work starts. Keep each guardrail short — a guardrail nobody reads is not a guardrail.

## In plain terms

It is easy to write a counter that agrees with `wc` on plain ASCII and is wrong on
the texts that the facts name — a carriage return, a no-break space, a file with
no final line feed, a byte that is not valid UTF-8, a file of 1 GiB; the defense is
a test for each of those cases, written from the facts before the code, and the
time, memory and coverage targets measured by the rules written below before
anyone looks at a result.

## 1. Pre-registered decisions — or the goalposts move

A decision rule chosen **after** seeing the result is a fitted parameter, not a
rule. Write the pass/fail numbers first, somewhere they cannot be quietly edited.

- **What must be pre-registered:** the three quality targets of the problem
  statement, with the measurement that the facts fix for each:
  - **Time:** a generated file of 10 MiB of ASCII text, with lines of about 80
    characters, is counted in at most 2 seconds of elapsed time of the whole
    process, from its start to its exit, on a computer with 4 cores and a
    solid-state disk, with the file already in the cache of the operating system
    (F-0001#9).
  - **Memory:** a generated file of 1 GiB is counted with a peak heap of at most
    64 MiB, sampled from `runtime.ReadMemStats` (`HeapAlloc`) every 10
    milliseconds during the count; the largest value counts, not the value at the
    end (F-0001#10).
  - **Coverage:** `go test -cover` reports at least 90 percent of the statements
    of the package `count`, which is apart from the package `main` (F-0001#11).
  - **Behaviour:** each rule of the behaviour section of the
    [problem statement](facts/problem-statement-brief.md) and each answer of
    [F-0001](facts/F-0001-setup-answers.md) is an acceptance bar with a test,
    and the success criteria of the delivery are those of F-0001#15.
- **Where the numbers freeze:** in the facts, which are immutable once committed,
  and in the requirements that cite them under [`prd/`](prd/). A test of a target
  states its number as a constant that cites its fact; it is not edited to make a
  run pass.
- **The bands, not a single line:** prefer **Pass / Investigate / Fail** to a
  single pass line on a noisy measure. Define "Investigate" with a rule written
  before you look — for example, one re-examination whose scope is fixed in
  advance; landing there twice counts as Fail.

## 2. Known pitfalls — the traps specific to this domain

The failure modes of counting text. For each: the trap, why it is silent, and the
check that catches it.

- ❌ **ASCII white space where Unicode `White_Space` is meant.** A word splitter
  that tests only space, tab and line feed agrees with the tests that use plain
  ASCII and miscounts a text with a no-break space (U+00A0) or an ideographic space
  (U+3000). It is silent because ASCII fixtures pass. **The check:** a unit test
  of the package `count` for each class of `White_Space` character, taken from the
  Unicode property, not from a hand list (`unicode.IsSpace` of Go follows it).
- ❌ **A byte count named a character count.** `len` of a Go string counts bytes,
  not code points, so `héllo` counts six. It is silent on ASCII text. **The
  check:** a test with characters of two, three and four bytes in UTF-8.
- ❌ **The last line without a line feed, and the carriage return.** Counting line
  feeds alone loses the last line of a file that does not end with one; stripping a
  carriage return before a line feed loses a character (F-0001#2). An empty file has
  zero lines, zero words and zero characters. **The check:** a test for each of
  these four texts: empty, `a`, `a⏎`, `a␍⏎`.
- ❌ **A boundary that splits a code point or a word.** A counter that reads in
  blocks, to keep the heap small, can split a multibyte code point or a word
  across two blocks and count it twice or call it invalid UTF-8. It is silent
  because small test files fit in one block. **The check:** a test that feeds the
  same text through a reader that returns one byte at a time, and compares the
  counts.
- ❌ **A whole file in memory.** Reading a file with `os.ReadFile` passes every
  small test and breaks the 64 MiB peak heap on a file of 1 GiB (F-0001#10). **The
  check:** the memory test samples the peak, not the heap at the end, because a
  collection before the end hides the peak.
- ❌ **A time measured on a cold cache or the wrong text.** A run on a cold cache,
  or on text that is not ASCII with lines of about 80 characters, measures
  something other than the target (F-0001#9). **The check:** the test generates its
  own file, reads it once to warm the cache, and measures the whole process.
- ❌ **An error that stops the run, or a row for a file that failed.** A file that
  cannot be read, a directory, or a file that is not valid UTF-8 gets a message on
  standard error, no row and nothing in the total, and the other files are still
  counted (F-0001#5, F-0001#6, F-0001#7). It is silent when the tests name one file
  only. **The check:** an end-to-end test with a good file, a missing file, a
  directory and an invalid file, which asserts the rows, the total, the messages
  and the exit code 1.
- ❌ **The order of the flags leaks into the output.** `tally -c -l` prints the
  lines before the characters (F-0001#3). **The check:** a test for each order of
  two flags.
- ❌ **The name `-` read as standard input.** Many commands read standard input for
  `-`; `tally` treats it as a file named `-` (F-0001#4). **The check:** a test that
  names `-`.

### Writing a lesson back (for every task — keep this)

A trap caught once should not be re-derived by the next task, so a lesson does not
stay on the issue that learned it. When a task ends, its author asks whether the task
taught a trap the next reader could hit; if it did, the lesson is written **here** as a
new `❌` pitfall — the trap, why it is silent, and the check that catches it — in the
same pull request. Gate step 7 asks the question, so the rule is applied rather than
merely written (see [Keeping documentation current](engineering-discipline.md#keeping-documentation-current)).

This is the one **cross-task** reach that the discipline adds on purpose.
[R6](issue-workflow.md#r6--agent-to-agent-communication-through-the-issue) and
[R7](issue-workflow.md#r7--decision-transparency-on-every-action) already keep the
coordination and the reasoning on the issue, and
[Honesty and evidence](engineering-discipline.md#honesty-and-evidence) already reports a
failure as a failure — but each is scoped to *one* issue thread. A lesson on issue #N is
discoverable only by someone who reads #N; §2 is where it reaches issue #N+1.

**The filter — or §2 grows until nobody reads it.** Write back only a trap that would
**catch the next reader**: a silent failure mode, a check that looked green for the
wrong reason, a footgun in the discipline or in counting text. Do **not** write back a one-off with
no general lesson, a restatement of a rule that already lives elsewhere, or the
blow-by-blow of the task — those belong to the issue thread and the commit history.
Volume is the failure mode here, not absence: a pitfall list nobody finishes reading
guards nothing.

### Gate pitfalls (for every task — keep these)

The gate is only as real as the thing that runs it. These traps let it report
success without having done its job.

- ❌ **An absolute `core.hooksPath`.** Worktrees **share** `.git/config`, so an
  absolute path binds every worktree to one checkout's hooks. A check added on a
  branch then does not run on that branch's own commits: the hook reports success
  having run something other than what the branch says it runs. It is silent
  because the hook still runs, still passes, and still uses the *right* files —
  only the *set of checks* comes from elsewhere. **The check:** install with a
  **relative** path, `git config core.hooksPath .githooks`, which git resolves per
  working tree. The `pre-commit` hook's own block 0 then refuses to run when the
  resolved hooks directory lies outside the tree being committed to, and
  [`.githooks/tests/provenance-check.sh`](../.githooks/tests/provenance-check.sh)
  proves it against real worktrees, reporting how many cases it ran rather than a
  count written down here to go stale.
  **A relative path escapes just as surely:** `../elsewhere/.githooks` is as
  foreign as any absolute one, so the check resolves the value instead of trusting
  that relative means local.
  **No path at all is the same trap:** with `core.hooksPath` unset git falls back
  to `.git/hooks`, which a linked worktree reaches through the shared *common* git
  directory. So block 0 judges the resolved directory whatever set it, and names
  the source it actually found — telling an operator to fix a setting they never
  set is its own dishonest report.
  **Bound on the damage:** CI invokes each check script directly and never through
  `core.hooksPath`, so this costs a local round trip, not a landed bug — a
  developer-experience gap, not an open gate.
- ❌ **A check that cannot fail.** A grep whose pattern also matches its own error
  message, a fixture harness that compares only exit codes, a coverage floor that
  counts zero as success. The check: for every assertion, make it fail on purpose
  once and read the reason — a green nobody attacked is not evidence.
- ❌ **A check that runs but does not block.** CI is green, the pull request
  merges, and nothing connects the two: no check is required on the default
  branch, so the green was a run result, not a merge control, and a red would
  have merged the same way. It is silent because the run result looks identical
  either way. The check:
  [make the checks required](ci/README.md#make-the-checks-required) on the
  default branch, and take the branch API read as the evidence — not the green run.

- ❌ **A check the change supplies is not a control.** CI checks out the pull
  request's own head and then runs the check from that checkout, so **the script
  that judges the change comes from the change**. Measured: a branch that replaces
  `docs/links/link-lint.sh` with `exit 0` passes that job — and replacing
  `docs/tests/run-discipline-tests.sh` as well turns **every** required job green
  over a dead link in the tree. Gutting a linter alone does not, because the
  fixture harness asserts exit codes and 19 `bad-*` cases stop failing; the runner
  is the single point.
  **The trap inside the remedy:** each script roots its scan at its own directory
  (`dirname $0`), so running the default branch's copy *where it sits* lints the
  wrong tree and reports OK. That was measured too, while building the fix.
  **The check:** every job in [`ci.yml`](../.github/workflows/ci.yml) restores the
  check scripts from the default branch **in place** before running them, so the
  branch's copy is never the judge.
  **Bound on the damage, and it is not nil:** a `pull_request` event runs the
  workflow as the branch has it, so a branch that edits `ci.yml` removes the
  restore step — closed only by review of `.github/**`, which wants a `CODEOWNERS`
  entry and a second human the forge knows about. Restoring also stops a bypass,
  not a merge: once a weakened check lands it *is* the default branch's copy. And
  a change that *improves* a check is judged by the older copy, so it lands in two
  steps. These checks are a control against forgetting, not against an operator
  who edits the check.

### Testing pitfalls (for every task — keep these)

These traps are not specific to counting text: they hurt every test suite, so the
Armature baseline wrote them out in full. Keep them; the traps of `tally` are above.

- ❌ **Testing after the code.** A test written to fit code that already "works"
  tends to encode the code's bugs as expected behaviour. The check: write the test
  first and watch it fail for the right reason
  ([strict TDD](engineering-discipline.md#requirements-traceability)).
- ❌ **Tests that depend on external state.** A test that reads a shared database, a
  live network, the wall clock, or another test's leftovers passes or fails for
  reasons unrelated to the code. The check: isolate and control every dependency,
  with a fresh fixture per run — see
  [`tests/scaling-checklist.md`](tests/scaling-checklist.md).
- ❌ **Tests that pass for the wrong reason.** A test that asserts nothing, asserts
  the wrong thing, or never actually exercises the path reports a safety that is not
  there — worse than no test. The check: confirm the test fails when the behaviour
  is broken; the red step is the proof.
- ❌ **Stale tests after a requirement changes.** When a requirement changes but its
  test does not, the suite now guards the old behaviour and blocks the new. The
  check: the [old-tests conflict rule](engineering-discipline.md#testing) — fix the
  code, update the requirement with a written reason, or retire the test; never
  weaken a passing old test.
- ❌ **Tests that slow down as the project grows.** A suite that creeps past the
  hook's patience gets skipped, and a skipped gate is no gate. The check: keep the
  cheap levels fast and cheap-first, push slow ones to CI, and bound each with
  the `-timeout` flag of `go test` — see [`tests/scaling-checklist.md`](tests/scaling-checklist.md).

### Reference-sweep pitfalls (for every task — keep these)

A change that edits references or a rule's wording across the tree has three silent
failure modes worth keeping.

- ❌ **A blanket search-and-substitute over a renamed record's citations.** When a record
  moves or a directory is renumbered, the same bare token can name *different*
  records in two places — a bare `ADR-0005` is the living `docs/adr/` record to one
  reader and the archived `docs/decisions/` one to another, because the two sequences
  once shared numbers. A global substitution of the token silently rewrites the citations
  you must **not** touch alongside the ones you must; and the reverse — a citation the
  sweep's pattern never matched (a compound like `ADR-0003/0005`, a token in a code
  span or a `.sh`/`.yml` comment, one split across a line break) — is silently *left*
  pointing at the wrong record. It is silent because **no linter catches it**:
  `link-lint` checks only that a *link* resolves, and a bare textual mention resolves
  to nothing, so a citation that now sends a reader to the wrong record still passes
  every check. **The check:** classify each occurrence by its **link target**, not its
  token — a link into `../adr/` is the living record and stays, a link into
  `../decisions/` is the archive and is rewritten — and read every *bare* mention by
  hand, in every token shape, since it carries no path to classify it. A
  pre-registered grep that must finish returning only the intended survivors (the
  mapping table and deliberate historical prose) is the closest thing to a gate; run
  it against the whole tree, not only the files you expected to touch.
- ❌ **A repoint that orphans a bare back-reference.** Repointing a citation can
  strand a *different* reference that named the target only through it. A comment
  reading `section 6 says …` leaned on a nearby `D-0003 section 6` for its antecedent;
  repoint every `D-0003 section 6` and the bare `section 6` is left pointing at a
  structure only the deleted record holds — wrong on the tree of this repository, and sharing
  **no token** with the thing you renamed. It is silent because a grep keyed on the
  obvious token (`D-000N`) cannot match a bare `section 6`, so the pre-registered
  check goes green over the survivor. **The check:** grep for the *shapes* a reference
  takes, not only the token — a bare `section N`, a `§`, a pronoun (`that section`,
  `the record`) whose antecedent you removed — and read the neighbourhood of every
  citation you changed, not the citation alone.
- ❌ **Editing a rule whose decision record is archived.** A rule lives in two places
  — its operative statement in a living doc, and the immutable decision record that
  first set it under `docs/decisions/`. Change the living one and the archived one
  still asserts the old, and **no check compares them** (`adr-lint` never reads
  `docs/decisions/`; `link-lint` checks resolution, not agreement). You cannot rewrite
  the immutable body to match; discharge the divergence with a `Status`-line
  **amendment pointer** on the archived record. **The check:** grep the whole tree —
  archive and forge templates included — for the old wording, and reconcile each living
  mirror or point each immutable one; a dated log entry recording history stays.
- ❌ **A hand-mirrored count or check-set that no linter guards.** The set of discipline
  linters — and how many there are — is spelled out by hand across many living
  docs, among them [`engineering-discipline.md`](engineering-discipline.md), this file,
  [`tests/test-levels.md`](tests/test-levels.md), the two `tests/README.md` files,
  [`ci/README.md`](ci/README.md) and
  [`.githooks/README.md`](../.githooks/README.md). Add or remove a check and every one
  can go stale, and a **removed** check leaves its name behind as a linter that no longer
  exists — a `link-lint` run stays green, because it resolves a *link*, not a claim. It is
  silent because the sentence still reads well and the count still looks deliberate: the
  Armature baseline once said `three`, `four` and `five` at once, and named an `agent-entry` linter that
  had been cut. **The check:** when you add or remove a discipline check, grep the whole
  tree for the check-set enumeration — the old name and each spelled count — and reconcile
  every living mirror in the same change; the immutable ADR and archived decision copies
  stay as history.

## 3. Validation — how you check you are not fooling yourself

A result is **untrusted** until it passes the checks below, and the pass is a
recorded event, not a memory. Order the checks cheap-first, so a failure stops the
expensive ones.

| # | Check | Pass condition | Cost |
|---|-------|----------------|------|
| 1 | `go vet ./...` and `gofmt -l .` | No report, and no file listed | seconds |
| 2 | `go test -short ./...` — the unit tests of the package `count` and the end-to-end tests of the command | Every test passes | seconds |
| 3 | `go test -cover ./count` | Every test passes, and the coverage is at least 90 percent of the statements | seconds |
| 4 | The time test on a generated file of 10 MiB | At most 2 seconds of elapsed time of the whole process | a minute |
| 5 | The memory test on a generated file of 1 GiB | A peak heap of at most 64 MiB | minutes |

Notes on how to read a failure: checks 1 to 3 catch a wrong count, a wrong row and
a wrong exit code, and are cheap enough for the `pre-commit` hook and CI. Checks 4
and 5 catch a counter that reads too much at once or works too slowly; they are
skipped under `-short` and run in CI and before a release, and after any change of
the package `count` that touches how it reads.

**The automated gate is this validation layer, mechanized.** The cheap, always-on
checks — the [discipline linters](engineering-discipline.md#testing) that this
repository ships (ADR, PRD and link) and their
[fixture self-tests](engineering-discipline.md#testing), the
[test levels](engineering-discipline.md#testing), lint, a security
scan, and the [commit-format](engineering-discipline.md#commit-messages)
check — run in the [`pre-commit` hook](engineering-discipline.md#git-hooks) for
fast local feedback and in [CI](engineering-discipline.md#continuous-integration)
as the authority. Treat those checks as pre-registered pass/fail rules under
section 1: they predate any single result and are not edited to make a change
pass. Wire the "cheap enough to wire into CI" checks from the table above into
both layers.

## 4. Mechanics

- **Frozen rules do not get edited.** Changing a guardrail after it is set means a
  new version with a written reason, the old one preserved. Legitimate reasons
  exist (a bug in the measure); silent edits do not.
- **This document holds the structure; the facts under [`facts/`](facts/) and the
  requirements under [`prd/`](prd/) hold the frozen values.** The values here mirror
  them, after the fact, never as the primary copy.

## Sources

- The [problem statement](facts/problem-statement-brief.md): the behaviour rules
  and the three quality targets.
- [F-0001](facts/F-0001-setup-answers.md): the answers that fix the line, the
  order of the counts, standard input, the total, the errors and the measurements
  (F-0001#2 to F-0001#11).
- The Unicode Standard, the property `White_Space` of `PropList.txt`.
