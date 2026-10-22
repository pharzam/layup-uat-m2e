# PRD-0001. tally, a command that counts the lines, words and characters of text files

Date: 2026-10-22

## 1. Header & "In plain terms"

| Field        | Value                                                             |
| ------------ | ---------------------------------------------------------------- |
| PRD ID       | `PRD-0001`                                                       |
| Title        | tally, a command that counts the lines, words and characters of text files |
| Status       | `Draft`                                                          |
| Date         | 2026-10-22                                                       |
| Author       | Agent session S-b476d863, task T-ut9g (issue #3)                 |
| Derives from | F-0003 (the problem statement), the answers Q-001 to Q-014 of the setup, and F-0002 — the [facts](../facts/) this PRD is built on |

### In plain terms

> `tally` is a small terminal command that tells a person how long a text file is:
> how many lines, words and characters it holds, for each file and in total. It
> exists so that someone who works in a terminal can get that answer quickly, even
> for a very large file.

## 2. Problem & evidence

A person who works in a terminal wants to know how long a text file is
(F-0003#2). `tally` reads the files that the person names and prints the number of
lines, words and characters of each, and a total (F-0003#3). It is written in Go,
with the standard library only (F-0003#1). What a line, a word and a character are
is fixed by the behaviour rules (F-0003#10, F-0003#11, F-0003#12) and the answers
Q-001 to Q-007; what the output looks like and how errors end the run is fixed by
F-0003#13 to F-0003#17. The quality targets — time, memory and coverage — are
F-0003#18 to F-0003#20, made measurable by Q-008 to Q-010. Every requirement below
also cites F-0002, the existing record of the setup.

## 3. Goals & non-goals

**Goals.**

- Count the lines, words and characters of each named file, or of standard input,
  and print a total for two files or more (REQ-001 to REQ-006).
- Report each file that cannot be counted on standard error, keep counting the
  others, and end with an exit code that says what happened (REQ-007 to REQ-012).
- Count by fixed definitions of a line, a word and a character (NFR-018 to NFR-020).
- Meet the time, memory and coverage targets of §8.

**Non-goals.** Other counts, such as bytes, paragraphs or sentences (F-0003#7,
REQ-028); unpacking archives or walking directories (F-0003#8, REQ-029); a
graphical interface, a network service or a plug-in system (F-0003#9, REQ-030).

## 4. Personas & jobs-to-be-done

- **The terminal user** — a person who works in a terminal and wants to know how
  long a text file is (F-0003#2). The job: get the lines, words and characters of
  one or more files, or of a pipe, in a form a script can read.

## 5. User stories

- As the terminal user, I name one or more files so that I see the lines, words and
  characters of each and a total (F-0003#3, F-0003#13).
- As the terminal user, I pass `-l`, `-w` or `-c` so that I see only the counts I
  want, always in the same order (F-0003#5, Q-002).
- As the terminal user, I pipe text into `tally` so that I count it without a file
  (F-0003#6, Q-003).
- As the terminal user, I see on standard error which file could not be counted and
  why, while the others are still counted, so that one bad file does not hide the
  rest (F-0003#14, F-0003#15, F-0003#16).
- As the terminal user, I read the exit code so that a script knows whether every
  file was counted (F-0003#17, Q-007).

## 6. Functional requirements
| REQ | Statement | MoSCoW | Phase | Facts |
| --- | --------- | ------ | ----- | ----- |
| REQ-001 | `tally` reports how long each text file that the user names is, as its number of lines, words and characters. |  |  | F-0003#2 F-0002 |
| REQ-002 | `tally` reads each file that the user names, prints for each file its number of lines, words and characters, and prints a total of those counts. |  |  | F-0003#3 F-0002 |
| REQ-003 | `tally` is invoked as `tally [-l] [-w] [-c] [file...]` and accepts zero or more file names. |  |  | F-0003#4 F-0002 |
| REQ-004 | `-l` selects the line count, `-w` the word count and `-c` the character count; the flags combine, and `tally` prints the selected counts in the fixed order lines, words, characters, whatever the order of the flags on the command line; with no flag, it prints all three counts. |  |  | F-0003#5 Q-002 F-0002 |
| REQ-005 | With no file name, `tally` reads and counts standard input, and the row for standard input ends after the counts, with no name; a file name `-` names a file called `-`, not standard input. |  |  | F-0003#6 Q-003 F-0002 |
| REQ-006 | The output has one row for each file counted without an error: the counts that the flags select, then the file name, each field separated from the next by exactly one space; with two files or more, a last row holds, in the same layout, the sum of each selected count over the files counted without an error, then the word `total`; a file that cannot be read or is not valid UTF-8 adds nothing to the total. |  |  | F-0003#13 Q-004 F-0002 |
| REQ-007 | When a named file cannot be read, `tally` writes a message to standard error that holds the file name and the reason, and ends with exit code 1. |  |  | F-0003#14 F-0002 |
| REQ-008 | A file that cannot be read does not stop the counting of the other named files. |  |  | F-0003#15 F-0002 |
| REQ-009 | A named directory is treated as a file that cannot be read: `tally` writes a message to standard error that holds its name and the reason, prints no row for it, adds nothing to the total, and ends with exit code 1. |  |  | F-0003#8 Q-006 F-0002 |
| REQ-010 | For a file that is not valid UTF-8, `tally` prints no row and adds nothing to the total, writes a message to standard error that holds the file name without a byte offset, still counts the remaining files, and ends with exit code 1. |  |  | F-0003#16 F-0003#21 Q-005 F-0002 |
| REQ-011 | When no error occurs, `tally` ends with exit code 0. |  |  | F-0003#17 F-0002 |
| REQ-012 | For an unknown flag or wrong usage, `tally` writes a usage message to standard error, counts nothing, and ends with exit code 2. |  |  | F-0003#17 Q-007 F-0002 |
| REQ-028 | Counting anything other than lines, words and characters, such as bytes, paragraphs or sentences. | Won't | — | F-0003#7 F-0002 |
| REQ-029 | Unpacking archives or walking directories. | Won't | — | F-0003#8 F-0002 |
| REQ-030 | A graphical interface, a network service or a plug-in system. | Won't | — | F-0003#9 F-0002 |

| Requirement | Acceptance criterion |
| ----------- | -------------------- |
| `REQ-001` | Given a text file of 3 lines, 6 words and 30 characters, running `tally` on it prints the counts 3, 6 and 30 for that file. |
| `REQ-002` | Given two text files with known counts, `tally` prints one row per file with its lines, words and characters, and a final row whose counts are the sums of the two files' counts. |
| `REQ-003` | Running `tally` with zero, one and three file names, with and without each of the flags `-l`, `-w` and `-c`, is accepted and produces counts; each named file appears in the output. |
| `REQ-004` | For a file with known counts, `tally -l` prints only the line count, `tally -w` only the word count, `tally -c` only the character count; `tally -c -l` and `tally -l -c` both print the line count then the character count; `tally -w -l -c` and `tally` with no flag both print the line, word and character counts in that order. |
| `REQ-005` | Piping a text with known counts into `tally` with no file name prints one row holding only the line, word and character counts of that text, with no trailing name, and the exit code is 0; running `tally -` in a directory that holds a file named `-` prints the counts of that file followed by the name `-`, and running it where no such file exists reports an unreadable file and exits with code 1. |
| `REQ-006` | For one file, the output is exactly one row of the form `<counts> <name>` with single spaces; for two files, the output is two file rows followed by a row `<sums> total`; with `-w`, each row holds only the word count and the name; for two readable files and one missing file, the total row holds the sums of the two readable files only. |
| `REQ-007` | Running `tally` on a file that does not exist, and on a file without read permission, writes to standard error a message holding that file name and the reason (such as no such file or permission denied), and the exit code is 1. |
| `REQ-008` | Running `tally` on a readable file, then a missing file, then a second readable file prints the rows of both readable files with correct counts, writes the error for the missing file to standard error, and exits with code 1. |
| `REQ-009` | Running `tally` on a readable file and a directory prints the row of the readable file and no row for the directory, writes to standard error a message holding the directory's name and the reason (such as is a directory), and the exit code is 1. |
| `REQ-010` | Running `tally` on a valid file, then a file that holds an invalid UTF-8 byte sequence (such as the byte 0xFF), then a second valid file prints the rows of both valid files and no row for the invalid one, writes to standard error a message holding that file name and no byte offset, and the exit code is 1. |
| `REQ-011` | Running `tally` on readable, valid UTF-8 files, and on valid standard input, ends with exit code 0. |
| `REQ-012` | Running `tally -x` on a readable file writes a usage message to standard error, prints nothing on standard output, and the exit code is 2. |
| `REQ-028` | The usage offers no flag for bytes, paragraphs or sentences, and no output row holds such a count (checked through NFR-014). |
| `REQ-029` | No code path unpacks an archive or walks a directory; a named directory is reported as REQ-009 states (checked through NFR-015). |
| `REQ-030` | The program opens no window and no listening socket and loads no plug-in (checked through NFR-016). |

## 7. Non-functional requirements
| REQ | Statement | MoSCoW | Phase | Facts |
| --- | --------- | ------ | ----- | ----- |
| NFR-013 | The product is written in Go, uses only the Go standard library, and is built with the Go toolchain. |  |  | F-0003#1 F-0002 |
| NFR-014 | The product counts only lines, words and characters, and offers no other count such as bytes, paragraphs or sentences. |  |  | F-0003#7 F-0002 |
| NFR-015 | The product unpacks no archive and walks no directory; a file is treated as not text exactly when it is not valid UTF-8. |  |  | F-0003#8 Q-006 F-0002 |
| NFR-016 | The product is a command-line program only, with no graphical interface, network service or plug-in system. |  |  | F-0003#9 F-0002 |
| NFR-017 | The functions that count the lines, the words and the characters of a reader live in a package `count` of the same Go module, separate from the package `main` of the command. |  |  | F-0003#20 Q-010 F-0002 |
| NFR-018 | A line is a run of characters that ends with a line feed; the last line of a file counts as a line when the file does not end with a line feed; a line feed alone ends an empty line, which counts as one line; a carriage return before a line feed is part of the line and counts as a character, like any other code point; an empty file has zero lines, zero words and zero characters. |  |  | F-0003#10 Q-001 F-0002 |
| NFR-019 | A word is a run of characters that are not white space, where white space is any character with the Unicode property `White_Space`. |  |  | F-0003#11 F-0002 |
| NFR-020 | A character is one Unicode code point of a text in UTF-8, whatever the number of bytes that encode it. |  |  | F-0003#12 F-0003#21 F-0002 |

| Requirement | Acceptance criterion |
| ----------- | -------------------- |
| `NFR-013` | The `go.mod` file declares no required module, every import is a standard-library package, and `go build` succeeds with the Go toolchain alone. |
| `NFR-014` | The usage accepts only the flags `-l`, `-w` and `-c`; any other flag, such as one for bytes, paragraphs or sentences, is rejected and no such count is printed. |
| `NFR-015` | No code path unpacks an archive or walks a directory; a file of bytes that is valid UTF-8 is counted whatever its content, and a file that is not valid UTF-8 is rejected as REQ-010 states. |
| `NFR-016` | The source imports no graphical, network-server or plug-in package (such as `net/http` or `plugin`), and the program opens no window and no listening socket. |
| `NFR-017` | The module holds a package `count`, distinct from the package `main`, that exports the functions counting lines, words and characters of an `io.Reader`, and the command imports it to do its counting. |
| `NFR-018` | The text `a\nb\n` counts 2 lines, `a\nb` counts 2 lines, `\n` counts 1 line and `\n\n` counts 2 lines; `a\r\nb\r\n` counts 2 lines and 6 characters; an empty file counts 0 lines, 0 words and 0 characters. |
| `NFR-019` | The text `one  two\tthree` counts 3 words; text separated by U+00A0 NO-BREAK SPACE or U+3000 IDEOGRAPHIC SPACE splits into separate words; a text of only white space counts 0 words. |
| `NFR-020` | The text `aé€😀` (1, 2, 3 and 4 bytes per code point) counts 4 characters; a combining sequence such as `e` followed by U+0301 counts 2 characters. |

## 8. Success metrics
| Metric | Fact |
| ------ | ---- |
| Time: counting a file of 10 MiB (10,485,760 bytes) of ASCII text with lines of about 80 characters takes at most 2 seconds of elapsed time of the whole process, from its start to its exit, on any computer with 4 cores and a solid-state disk, with the file already in the cache of the operating system. Measured by a test that generates the file, reads it once to warm the cache, then runs `tally` on it and measures the elapsed time. | F-0003#18 F-0003#22 Q-008 F-0002 |
| Memory: counting a file of 1 GiB (1,073,741,824 bytes) uses at most 64 MiB (67,108,864 bytes) of peak heap during the run. Measured by a test that samples the heap in use (`runtime.ReadMemStats`, `HeapAlloc`) every 10 milliseconds during the count and takes the largest sampled value. | F-0003#19 F-0003#22 F-0003#23 Q-009 F-0002 |
| Coverage: the tests of the package `count` cover at least 90 percent of its statements. Measured by the output of `go test -cover` for the package `count`, which reports at least 90.0 percent; the coverage of the package `main` is not part of the target. | F-0003#20 Q-010 F-0002 |
| Delivery: `tally` prints the lines, words and characters of each named file and a total, as the statement's behaviour rules say. Measured by the acceptance criteria of REQ-001 to REQ-006. | Q-014 F-0002 |
| Delivery: a file that cannot be read or is not valid UTF-8 gives a message and the exit code 1 while the other files are still counted. Measured by the acceptance criteria of REQ-007 to REQ-010. | Q-014 F-0002 |
| Delivery: a 10 MiB file is counted in at most 2 seconds and a 1 GiB file with at most 64 MiB of peak heap. Measured by the time and memory metrics above. | Q-014 F-0002 |
| Delivery: the package `count` has at least 90 percent statement coverage. Measured by the coverage metric above. | Q-014 F-0002 |

## 9. Rollout & phases
| Phase | Name | Ships |
| ----- | ---- | ----- |
| 1 | First release of `tally` | The command `tally [-l] [-w] [-c] [file...]` and its package `count`: every requirement that the bet gives phase 1, meeting the time, memory and coverage targets of §8. |

A `Won't` row has the phase `—`: it never ships.

## 10. Risks

- Counting text has silent traps — Unicode white space, code points against bytes,
  the last line and the carriage return, a block boundary inside a code point. Each
  is a pitfall of [`../guardrails.md` §2](../guardrails.md#2-known-pitfalls--the-traps-specific-to-this-domain),
  with the check that catches it.
- A whole file read into memory breaks the memory target, and a time measured on a
  cold cache measures the wrong thing; the targets and their measurements are
  pre-registered in [`../guardrails.md` §1](../guardrails.md#1-pre-registered-decisions--or-the-goalposts-move).
- An error that stops the run, or a row printed for a file that failed, is silent
  when a test names one file only ([§2](../guardrails.md#2-known-pitfalls--the-traps-specific-to-this-domain)).

## 11. Open questions & assumptions

- No question is open: the answers Q-001 to Q-010 settled the questions the problem
  statement left, and Q-014 states the success criteria of the delivery.
- The MoSCoW priority and the phase of REQ-001 to REQ-012 and NFR-013 to NFR-020
  are set at the bet; this draft leaves those cells empty on purpose. The three
  `Won't` rows REQ-028 to REQ-030 hold the non-goals of §3.
- The F-0003 record of the problem statement is rendered from the recorded spans at
  the first bet; until then F-0003#n names fact n of those spans, and Q-NNN names
  an answer of the setup.
- The draft criterion of NFR-015 cited the rejection of an invalid UTF-8 file as
  "R-009"; R-009 is the directory rule, and the rejection is REQ-010. This PRD
  cites REQ-010.

## 12. Requirements traceability matrix

One row per requirement in §6 / §7. This is the line a reader follows from the
customer's words to the test that proves them. No ADR of `tally` exists yet, and no
test is written yet, so those cells are `—`.

| REQ | Facts | Guardrail | ADR | Task | Test |
| --- | ----- | --------- | --- | ---- | ---- |
| REQ-001 | F-0003#2 F-0002 | — | — | T-ut9g | — |
| REQ-002 | F-0003#3 F-0002 | — | — | T-ut9g | — |
| REQ-003 | F-0003#4 F-0002 | — | — | T-ut9g | — |
| REQ-004 | F-0003#5 Q-002 F-0002 | [§2 the order of the flags](../guardrails.md#2-known-pitfalls--the-traps-specific-to-this-domain) | — | T-ut9g | — |
| REQ-005 | F-0003#6 Q-003 F-0002 | [§2 the name `-` read as standard input](../guardrails.md#2-known-pitfalls--the-traps-specific-to-this-domain) | — | T-ut9g | — |
| REQ-006 | F-0003#13 Q-004 F-0002 | [§2 an error that stops the run, or a row for a file that failed](../guardrails.md#2-known-pitfalls--the-traps-specific-to-this-domain) | — | T-ut9g | — |
| REQ-007 | F-0003#14 F-0002 | [§2 an error that stops the run, or a row for a file that failed](../guardrails.md#2-known-pitfalls--the-traps-specific-to-this-domain) | — | T-ut9g | — |
| REQ-008 | F-0003#15 F-0002 | [§2 an error that stops the run, or a row for a file that failed](../guardrails.md#2-known-pitfalls--the-traps-specific-to-this-domain) | — | T-ut9g | — |
| REQ-009 | F-0003#8 Q-006 F-0002 | [§2 an error that stops the run, or a row for a file that failed](../guardrails.md#2-known-pitfalls--the-traps-specific-to-this-domain) | — | T-ut9g | — |
| REQ-010 | F-0003#16 F-0003#21 Q-005 F-0002 | [§2 an error that stops the run, or a row for a file that failed](../guardrails.md#2-known-pitfalls--the-traps-specific-to-this-domain) | — | T-ut9g | — |
| REQ-011 | F-0003#17 F-0002 | — | — | T-ut9g | — |
| REQ-012 | F-0003#17 Q-007 F-0002 | — | — | T-ut9g | — |
| REQ-028 | F-0003#7 F-0002 | — | — | T-ut9g | — |
| REQ-029 | F-0003#8 F-0002 | — | — | T-ut9g | — |
| REQ-030 | F-0003#9 F-0002 | — | — | T-ut9g | — |
| NFR-013 | F-0003#1 F-0002 | — | — | T-ut9g | — |
| NFR-014 | F-0003#7 F-0002 | — | — | T-ut9g | — |
| NFR-015 | F-0003#8 Q-006 F-0002 | — | — | T-ut9g | — |
| NFR-016 | F-0003#9 F-0002 | — | — | T-ut9g | — |
| NFR-017 | F-0003#20 Q-010 F-0002 | [§1 coverage of the package `count`](../guardrails.md#1-pre-registered-decisions--or-the-goalposts-move) | — | T-ut9g | — |
| NFR-018 | F-0003#10 Q-001 F-0002 | [§2 the last line without a line feed, and the carriage return](../guardrails.md#2-known-pitfalls--the-traps-specific-to-this-domain) | — | T-ut9g | — |
| NFR-019 | F-0003#11 F-0002 | [§2 ASCII white space where Unicode `White_Space` is meant](../guardrails.md#2-known-pitfalls--the-traps-specific-to-this-domain) | — | T-ut9g | — |
| NFR-020 | F-0003#12 F-0003#21 F-0002 | [§2 a byte count named a character count; a boundary that splits a code point](../guardrails.md#2-known-pitfalls--the-traps-specific-to-this-domain) | — | T-ut9g | — |

## 13. Change log

| Date       | Change                     | Requirement(s) affected |
| ---------- | -------------------------- | ----------------------- |
| 2026-10-22 | First draft, derived from the revised draft of T-ut9g: REQ-001 to REQ-012 and NFR-013 to NFR-020 keep the numbers of the draft rows R-001 to R-020; R-021 to R-027 are the success metrics of §8; REQ-028 to REQ-030 are added as the `Won't` rows of the three non-goals. | REQ-001 to REQ-012, REQ-028 to REQ-030, NFR-013 to NFR-020 |
| 2026-10-22 | The criterion of NFR-015 cites REQ-010 for the rejection of an invalid UTF-8 file, where the draft cited R-009, the directory rule. | NFR-015 |
