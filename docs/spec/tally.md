# Specification of tally

The specification of each requirement of [PRD-0001](../prd/PRD-0001-tally.md),
one section per ID. The PRD holds the statement, the facts and the acceptance
criterion of each; this file says what the behaviour means in detail, so a build
task can write the test first. F-0003#n is fact n of the
[problem statement](../facts/problem-statement-brief.md); Q-NNN is an answer of
the operators.

## REQ-001 — Report the length of each named file

For each text file that the user names, `tally` reports how long it is as three
counts: lines, words and characters, each as NFR-018, NFR-019 and NFR-020 define
them. A file of 3 lines, 6 words and 30 characters
gives the counts 3, 6 and 30 (F-0003#2).

## REQ-002 — Count each file and print a total

`tally` reads every named file, prints the counts of each, and prints a total of
those counts. The total follows the layout and the rule of REQ-006 (F-0003#3).

## REQ-003 — The command line

The command is `tally [-l] [-w] [-c] [file...]`. It accepts zero, one or more
file names, with or without any of the three flags; each named file is counted or
reported (F-0003#4).

## REQ-004 — Select and order the counts

`-l` selects lines, `-w` words and `-c` characters. Flags combine. The selected
counts always print in the order lines, words, characters, whatever the order of
the flags; `tally -c -l` and `tally -l -c` print the same row. With no flag, all
three counts print (F-0003#5, Q-002).

## REQ-005 — Standard input and the name `-`

With no file name, `tally` counts standard input; its row holds the counts only,
with no name after them, and the exit code is 0 when no error occurs. The name
`-` is an ordinary file name: `tally -` counts a file called `-`, and reports an
unreadable file when no such file exists (F-0003#6, Q-003).

## REQ-006 — The layout of the output

One row per file counted without an error: the selected counts, then the file
name, each field separated from the next by exactly one space. With two named
files or more, a last row holds the sum of each selected count over the files
counted without an error, then the word `total`, in the same layout. A file that
cannot be read, a directory, or a file that is not valid UTF-8 adds nothing to the
total (F-0003#13, Q-004).

## REQ-007 — A file that cannot be read

When a named file cannot be read — it does not exist, or permission is denied —
`tally` writes to standard error a message that holds the file name and the
reason, prints no row for it, and ends with exit code 1 (F-0003#14).

## REQ-008 — The other files are still counted

An unreadable file does not stop the run: every other named file is still
counted and printed in order, and the exit code is 1 at the end (F-0003#15).

## REQ-009 — A directory

A named directory is treated as a file that cannot be read: a message on standard
error with its name and the reason (such as "is a directory"), no row, nothing
added to the total, and exit code 1. `tally` never walks into it (F-0003#8,
Q-006).

## REQ-010 — A file that is not valid UTF-8

A file that holds a byte sequence that is not valid UTF-8, such as the byte 0xFF,
gets no row and adds nothing to the total. `tally` writes to standard error a
message that holds the file name and no byte offset, counts the remaining files,
and ends with exit code 1 (F-0003#16, F-0003#21, Q-005).

## REQ-011 — Exit code 0

When every named file, or standard input, is counted without an error, the exit
code is 0 (F-0003#17).

## REQ-012 — Wrong usage

An unknown flag, such as `-x`, or wrong usage writes a usage message to standard
error, prints nothing on standard output, counts nothing, and ends with exit code
2 (F-0003#17, Q-007).

## NFR-013 — Go and its standard library

The product is written in Go and built with the Go toolchain alone. `go.mod`
declares no required module, and every import is a package of the standard
library (F-0003#1).

## NFR-014 — Only three counts

The product counts lines, words and characters, and nothing else: no flag for
bytes, paragraphs or sentences exists, and such a flag is wrong usage as REQ-012
states (F-0003#7).

## NFR-015 — No archives, no directory walks, text is valid UTF-8

No code path unpacks an archive or walks a directory. A file is text exactly when
it is valid UTF-8: such a file is counted whatever its content, and any other file
is rejected as REQ-010 states (F-0003#8, Q-006).

## NFR-016 — A command-line program only

The product has no graphical interface, network service or plug-in system: it
imports no such package (such as `net/http` or `plugin`), and opens no window and
no listening socket (F-0003#9).

## NFR-017 — The package `count`

The functions that count the lines, the words and the characters of an
`io.Reader` live in a package `count` of the same Go module, separate from the
package `main`; the command imports `count` to do its counting (F-0003#20,
Q-010).

## NFR-018 — What a line is

A line is a run of characters that ends with a line feed. The last line counts
when the file does not end with a line feed. A line feed alone ends an empty line,
which counts as one. A carriage return before a line feed is part of the line and
counts as a character. An empty file has 0 lines, 0 words and 0 characters. So
`a\nb\n` and `a\nb` count 2 lines, `\n` 1 line, `\n\n` 2 lines, and `a\r\nb\r\n`
2 lines and 6 characters (F-0003#10, Q-001).

## NFR-019 — What a word is

A word is a run of characters that are not white space, where white space is any
character with the Unicode property `White_Space`, including U+00A0 and U+3000.
`one  two\tthree` counts 3 words; a text of only white space counts 0 words
(F-0003#11).

## NFR-020 — What a character is

A character is one Unicode code point of the UTF-8 text, whatever the number of
bytes that encode it: `aé€😀` counts 4 characters, and `e` followed by U+0301
counts 2. A code point whose bytes straddle the boundary of two reads still counts
once (F-0003#12, F-0003#21).
