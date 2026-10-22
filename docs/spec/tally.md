# The specification of tally

In plain terms: one section for each requirement of
[PRD-0001](../prd/PRD-0001-tally.md), saying what `tally` does for it and how a
test shows it. The PRD holds the statement, the facts and the priority of each
requirement; this file explains it. Where the two disagree, the PRD wins, and the
disagreement is a defect to fix.

## REQ-001 — Report how long a file is

For each text file that the user names, `tally` reports its length as three
numbers: its lines, its words and its characters, as NFR-018 to NFR-020 define
them. A file of 3 lines, 6 words and 30 characters gives 3, 6 and 30.

## REQ-002 — A row for each file, and a total

`tally` reads every file that the user names, in the order given, and prints its
counts on a row of its own. After the rows of the files, it prints a total of the
counts. The layout of the rows and when the total appears are REQ-006.

## REQ-003 — The command line

The command is `tally [-l] [-w] [-c] [file...]`. Each flag is optional, and any
number of file names follows, zero included. Each named file appears in the
output, unless it fails (REQ-007 to REQ-010).

## REQ-004 — The flags select the counts, in a fixed order

`-l` selects the lines, `-w` the words and `-c` the characters. The flags combine.
The selected counts always come in the order lines, words, characters, whatever
the order of the flags: `tally -c -l` and `tally -l -c` print the same row. With
no flag, all three counts are printed.

## REQ-005 — Standard input, and the name `-`

With no file name, `tally` counts standard input. Its row ends after the counts,
with no name. The name `-` is not standard input: it names a file called `-`, which
is counted like any other file, or reported as unreadable when it does not exist.

## REQ-006 — The layout of the output

A row holds the selected counts, then the file name, each separated from the next
by exactly one space: `<counts> <name>`. A file that fails gets no row. With two
files or more, a last row holds the sum of each selected count over the files
counted without an error, in the same layout, followed by the word `total`. A
file that cannot be read or is not valid UTF-8 adds nothing to the total.

## REQ-007 — A file that cannot be read

When a named file cannot be read — it does not exist, or the user may not read
it — `tally` writes a message to standard error with the file name and the reason,
prints no row for it, and ends with the exit code 1.

## REQ-008 — The other files are still counted

A file that cannot be read never stops the run. `tally` reports it and goes on to
the next file, so the rows of the readable files are printed with their correct
counts, and the exit code at the end is 1.

## REQ-009 — A directory is a file that cannot be read

A named directory is not walked. It is a file that cannot be read: a message on
standard error with its name and the reason (such as "is a directory"), no row,
nothing in the total, and the exit code 1. The other files are still counted.

## REQ-010 — A file that is not valid UTF-8

A file that holds a byte sequence that is not valid UTF-8, such as the byte 0xFF,
is skipped entirely, like an unreadable file: no row, nothing in the total, a
message on standard error with its name and no byte offset, and the exit code 1.
The remaining files are still counted.

## REQ-011 — Exit code 0 when no error occurs

When every named file, or standard input, is counted without an error, `tally`
ends with the exit code 0.

## REQ-012 — Wrong usage

An unknown flag, such as `-x`, or any other wrong usage, prints a usage message on
standard error and nothing on standard output. `tally` counts nothing and ends with
the exit code 2.

## REQ-028 — Won't: other counts

`tally` counts lines, words and characters, and nothing else: no bytes, no
paragraphs, no sentences. This is a non-goal of the product; NFR-014 is the
constraint that keeps it.

## REQ-029 — Won't: archives and directories

`tally` unpacks no archive and walks no directory. A directory that the user names
is reported as REQ-009 says. This is a non-goal of the product; NFR-015 is the
constraint that keeps it.

## REQ-030 — Won't: an interface other than the command line

`tally` has no graphical interface, no network service and no plug-in system. This
is a non-goal of the product; NFR-016 is the constraint that keeps it.

## NFR-013 — Go and its standard library

`tally` is written in Go, imports only packages of the Go standard library, and is
built with the Go toolchain alone. Its `go.mod` requires no module.

## NFR-014 — Only three counts

The only flags are `-l`, `-w` and `-c`. A flag for any other count, such as bytes,
paragraphs or sentences, is an unknown flag and gets the usage error of REQ-012.

## NFR-015 — No archive, no directory walk; text means valid UTF-8

No code path unpacks an archive or walks a directory. Whether a file is text
depends only on whether its bytes are valid UTF-8: a valid file is counted whatever
its content, and an invalid one is rejected as REQ-010 states.

## NFR-016 — A command-line program only

The source imports no graphical, network-server or plug-in package, such as
`net/http` or `plugin`. The program opens no window and no listening socket.

## NFR-017 — The package `count`

The functions that count the lines, the words and the characters of an
`io.Reader` live in a package `count` of the same Go module, apart from the
package `main` of the command, which imports it. The coverage target of PRD-0001
§8 covers the package `count` only.

## NFR-018 — What a line is

A line is a run of characters that ends with a line feed. The last line counts even
when the file does not end with a line feed, so `a\nb` has 2 lines. A line feed
alone ends an empty line, which counts: `\n\n` has 2 lines. A carriage return
before a line feed is part of the line and counts as a character: `a\r\nb\r\n` has
2 lines and 6 characters. An empty file has 0 lines, 0 words and 0 characters.

## NFR-019 — What a word is

A word is a run of characters that are not white space. White space is any
character with the Unicode property `White_Space`, not only the ASCII space, tab
and line feed: a no-break space (U+00A0) or an ideographic space (U+3000) splits
two words. A text of only white space has 0 words.

## NFR-020 — What a character is

A character is one Unicode code point of the UTF-8 text, whatever the number of
bytes that encode it: `aé€😀` has 4 characters in 10 bytes. A combining sequence,
such as `e` followed by U+0301, is 2 characters. A counter that reads in blocks
must not split a code point across two blocks.
