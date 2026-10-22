# PRD-0001. tally — count the lines, words and characters of text files

Date: 2026-10-22

## 1. Header & "In plain terms"

| Field        | Value                                                             |
| ------------ | ---------------------------------------------------------------- |
| PRD ID       | `PRD-0001`                                                       |
| Title        | tally — a Go command that counts the lines, words and characters of text files |
| Status       | `Draft`                                                          |
| Date         | 2026-10-22                                                        |
| Author       | Claude Opus 5.5, task T-ut9g (issue #3)                           |
| Derives from | F-0003 (the problem statement), F-0002, and the answers Q-001 to Q-014 |

### In plain terms

> `tally` is a small terminal command that tells a person how long a text file
> is: how many lines, words and characters it holds, file by file, with a total.
> It exists because a person who works in a terminal wants that answer quickly
> and exactly.

## 2. Problem & evidence

A person who works in a terminal wants to know how long a text file is
(F-0003#2). The product reads the files that the person names and prints the
number of lines, words and characters of each, and a total (F-0003#3). It is
written in Go with the standard library only (F-0003#1). The rules of counting
are fixed by the statement (F-0003#10 to F-0003#12) and sharpened by the answers
Q-001 to Q-007; the quality targets are F-0003#18 to F-0003#20, sharpened by
Q-008 to Q-010. The facts F-0003#n are the numbered spans of the
[problem statement](../facts/problem-statement-brief.md); the answers Q-NNN are
the operators' answers of issue #1.

## 3. Goals & non-goals

**Goals.**

- Count the lines, words and characters of each named file, or of standard
  input, and print a total for two files or more (REQ-001 to REQ-006).
- Report each file that cannot be counted without stopping the others, with an
  exit code that tells success from failure (REQ-007 to REQ-012).
- Meet the time, memory and coverage targets of §8.

**Non-goals.** `tally` offers no other count, such as bytes, paragraphs or
sentences (F-0003#7); it does not unpack archives or walk directories, and it
rejects a file that is not valid UTF-8 rather than guess its encoding (F-0003#8);
it has no graphical interface, network service or plug-in system (F-0003#9).
NFR-014, NFR-015 and NFR-016 hold this boundary.

## 4. Personas & jobs-to-be-done

- **Terminal user.** A person who works in a terminal and wants to know how long
  a text file is (F-0003#2).

## 5. User stories

- As the terminal user, I run `tally` on the files I name so that I see the
  lines, words and characters of each, and a total (F-0003#3, F-0003#4).
- As the terminal user, I pick only the counts I need with `-l`, `-w` and `-c`
  (F-0003#5, Q-002).
- As the terminal user, I pipe text into `tally` so that I count text that is
  not in a file (F-0003#6, Q-003).
- As the terminal user, I learn which file could not be counted, and why, while
  the other files are still counted (F-0003#14 to F-0003#16).

## 6. Functional requirements

| REQ | Statement | MoSCoW | Phase | Facts |
| --- | --------- | ------ | ----- | ----- |
| REQ-001 | `tally` reports how long each text file that the user names is, as its number of lines, words and characters. |  |  | F-0003#2 F-0002 |
| REQ-002 | `tally` reads each named file, prints its number of lines, words and characters, and prints a total of those counts. |  |  | F-0003#3 F-0002 |
| REQ-003 | The command is invoked as `tally [-l] [-w] [-c] [file...]` and accepts zero or more file names. |  |  | F-0003#4 F-0002 |
| REQ-004 | `-l` selects the line count, `-w` the word count and `-c` the character count; the flags combine, the selected counts print in the fixed order lines, words, characters whatever the order of the flags, and with no flag all three print. |  |  | F-0003#5 Q-002 F-0002 |
| REQ-005 | With no file name, `tally` counts standard input, and its row ends after the counts, with no name; the name `-` names a file called `-`, not standard input. |  |  | F-0003#6 Q-003 F-0002 |
| REQ-006 | The output has one row per file counted without an error: the selected counts, then the file name, each field separated by exactly one space; with two files or more, a last row holds, in the same layout, the sum of each selected count over the files counted without an error, then the word `total`; a file that cannot be read or is not valid UTF-8 adds nothing to the total. |  |  | F-0003#13 Q-004 F-0002 |
| REQ-007 | When a named file cannot be read, `tally` writes a message to standard error that holds the file name and the reason, and ends with exit code 1. |  |  | F-0003#14 F-0002 |
| REQ-008 | A file that cannot be read does not stop the counting of the other named files. |  |  | F-0003#15 F-0002 |
| REQ-009 | A named directory is a file that cannot be read: `tally` writes a message to standard error with its name and the reason, prints no row for it, adds nothing to the total, and ends with exit code 1. |  |  | F-0003#8 Q-006 F-0002 |
| REQ-010 | For a file that is not valid UTF-8, `tally` prints no row, adds nothing to the total, writes a message to standard error that holds the file name and no byte offset, still counts the remaining files, and ends with exit code 1. |  |  | F-0003#16 F-0003#21 Q-005 F-0002 |
| REQ-011 | When no error occurs, `tally` ends with exit code 0. |  |  | F-0003#17 F-0002 |
| REQ-012 | For an unknown flag or wrong usage, `tally` writes a usage message to standard error, counts nothing, and ends with exit code 2. |  |  | F-0003#17 Q-007 F-0002 |

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

## 7. Non-functional requirements

| REQ | Statement | MoSCoW | Phase | Facts |
| --- | --------- | ------ | ----- | ----- |
| NFR-013 | The product is written in Go, uses only the Go standard library, and is built with the Go toolchain. |  |  | F-0003#1 F-0002 |
| NFR-014 | The product counts only lines, words and characters, and offers no other count such as bytes, paragraphs or sentences. |  |  | F-0003#7 F-0002 |
| NFR-015 | The product does not unpack archives or walk directories; a file is not text exactly when it is not valid UTF-8. |  |  | F-0003#8 Q-006 F-0002 |
| NFR-016 | The product is a command-line program only, with no graphical interface, network service or plug-in system. |  |  | F-0003#9 F-0002 |
| NFR-017 | The functions that count the lines, the words and the characters of a reader live in a package `count` of the same Go module, separate from the package `main` of the command. |  |  | F-0003#20 Q-010 F-0002 |
| NFR-018 | A line is a run of characters that ends with a line feed; the last line counts when the file does not end with a line feed; a line feed alone ends an empty line, which counts as one line; a carriage return before a line feed is part of the line and counts as a character; an empty file has zero lines, zero words and zero characters. |  |  | F-0003#10 Q-001 F-0002 |
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
| Counting a file of 10 MiB (10,485,760 bytes) of ASCII text with lines of about 80 characters, already in the cache of the operating system, takes at most 2 seconds of elapsed time of the whole process, from its start to its exit, on a computer with 4 cores and a solid-state disk; measured by a test that generates the file, reads it once to warm the cache, then runs `tally` on it. | F-0003#18 F-0003#22 Q-008 F-0002 |
| Counting a file of 1 GiB (1,073,741,824 bytes) uses at most 64 MiB (67,108,864 bytes) of peak heap; measured by a test that samples the heap in use (`runtime.ReadMemStats`, `HeapAlloc`) every 10 milliseconds during the count and takes the largest value. | F-0003#19 F-0003#22 F-0003#23 Q-009 F-0002 |
| The tests of the package `count` cover at least 90 percent of its statements, as `go test -cover` reports for the package `count` (at least 90.0 percent); the package `main` is not part of the target. | F-0003#20 Q-010 F-0002 |
| The delivery's success criterion: `tally` prints the lines, words and characters of each named file and a total, as the statement's behaviour rules say. | Q-014 F-0002 |
| The delivery's success criterion: a file that cannot be read or is not valid UTF-8 gives a message and the exit code 1 while the other files are still counted. | Q-014 F-0002 |
| The delivery's success criterion: a 10 MiB file is counted in at most 2 seconds and a 1 GiB file with at most 64 MiB of peak heap. | Q-014 F-0002 |
| The delivery's success criterion: the package `count` has at least 90 percent statement coverage. | Q-014 F-0002 |

## 9. Rollout & phases

| Phase | Name | Ships |
| ----- | ---- | ----- |
| 1 | First release | The command `tally` with its flags, standard input, the output rows and the total, the error handling and exit codes, the package `count`, and the targets of §8. |

## 10. Risks

- Counting text has traps — the last line without a line feed, a carriage
  return, Unicode white space, a code point split across a read boundary. The
  pitfalls of [`../guardrails.md`](../guardrails.md) §2 hold the line.
- Reading a whole file into memory breaks the peak-heap target for 1 GiB; the
  counting must stream.

## 11. Open questions & assumptions

- The draft criterion of NFR-015 cited R-009 (the directory rule) for the
  rejection of a file that is not valid UTF-8; that rule is REQ-010, and the
  criterion here cites REQ-010.
- The MoSCoW priority and the phase of each requirement are set at the bet.

## 12. Requirements traceability matrix

One row per requirement in §6 / §7. This is the line a reader follows from the
customer's words to the test that proves them.

| REQ | Facts | Guardrail | ADR | Task | Test |
| --- | ----- | --------- | --- | ---- | ---- |
| REQ-001 | F-0003#2 F-0002 | — | — | — | — |
| REQ-002 | F-0003#3 F-0002 | — | — | — | — |
| REQ-003 | F-0003#4 F-0002 | — | — | — | — |
| REQ-004 | F-0003#5 Q-002 F-0002 | — | — | — | — |
| REQ-005 | F-0003#6 Q-003 F-0002 | — | — | — | — |
| REQ-006 | F-0003#13 Q-004 F-0002 | — | — | — | — |
| REQ-007 | F-0003#14 F-0002 | — | — | — | — |
| REQ-008 | F-0003#15 F-0002 | — | — | — | — |
| REQ-009 | F-0003#8 Q-006 F-0002 | — | — | — | — |
| REQ-010 | F-0003#16 F-0003#21 Q-005 F-0002 | — | — | — | — |
| REQ-011 | F-0003#17 F-0002 | — | — | — | — |
| REQ-012 | F-0003#17 Q-007 F-0002 | — | — | — | — |
| NFR-013 | F-0003#1 F-0002 | — | — | — | — |
| NFR-014 | F-0003#7 F-0002 | — | — | — | — |
| NFR-015 | F-0003#8 Q-006 F-0002 | — | — | — | — |
| NFR-016 | F-0003#9 F-0002 | — | — | — | — |
| NFR-017 | F-0003#20 Q-010 F-0002 | — | — | — | — |
| NFR-018 | F-0003#10 Q-001 F-0002 | — | — | — | — |
| NFR-019 | F-0003#11 F-0002 | — | — | — | — |
| NFR-020 | F-0003#12 F-0003#21 F-0002 | — | — | — | — |

## 13. Change log

| Date       | Change                     | Requirement(s) affected |
| ---------- | -------------------------- | ----------------------- |
| 2026-10-22 | First draft, from the revised draft of task T-ut9g; the NFR-015 criterion cites REQ-010, not R-009. | REQ-001 to REQ-012, NFR-013 to NFR-020 |
