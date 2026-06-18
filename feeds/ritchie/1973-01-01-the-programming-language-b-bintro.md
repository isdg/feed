---
title: "The Programming Language B"
url: https://cm-bell-labs.github.io/who/dmr/bintro.html
published: "1973-01-01T00:00:00Z"
feed: ritchie
guid: https://cm-bell-labs.github.io/who/dmr/bintro.html
---

# The Programming Language B

THE PROGRAMMING LANGUAGE B

S. C. Johnson

B. W. Kernighan

Bell Laboratories

Murray Hill, New Jersey

##### ABSTRACT

B is a computer language designed by D. M. Ritchie and K. L.
Thompson, for primarily non-numeric applications such as system
programming. These typically involve complex logical
decision-making, and processing of integers, characters, and bit strings.
On the H6070 TSS system, B programs are usually much easier to
write and understand than assembly language programs, and object
code efficiency is almost as good. Implementation of simple TSS
subsystems is an especially appropriate use for B.
This technical report contains a description of the MH-TSS
(Honeywell 6070) version of B (by S. C. Johnson), and a tutorial
introduction to most of the features of the language (by B. W.
Kernighan).

DMR note, June 1997:

This WWW page is a rendition of
Bell Laboratories Computing Science Technical
Report #8: The Programming Language B, January, 1973.
It was scanned using Adobe OCR software and
the version here was edited by Dennis Ritchie.
It is divided into two sections, each in several formats:

- A Tutorial Introduction
to the Language B,
by Brian Kernighan, is browsable HTML; also available as
PostScript or
PDF.

- User's Reference to B on MH-TSS,
by Steve Johnson, is browsable HTML; also available in
PostScript or PDF.

For scholars, the page images are also available:

- PDF Tutorial is a scanned PDF image of
the tutorial. Caution: 1.2MB in size.
- PDF Reference is a scanned PDF image of
the reference. Caution: 1.4MB in size.

The document seems to exist only on (partially) original
paper printed on a Teletype model 37 terminal. It uses underlining
for emphasis. You need to look at the PDF scans to verify any
typos I might have introduced in cleaning up the OCR, which was
pretty good except where there was underlining or double-quote
characters; they tended to merge into the line above. I
avoided the urge to redact the original except for a few
obvious mistakes, in particular some missing semicolons
in the syntax for some of the commands.

When this CSTR was issued, which was probably
some months after the papers were written,
the use of B was growing
on the local Honeywell GCOS system.
Its time-sharing facility was
called MH-TSS here, and it was then the main computation
facility at Bell Labs in Murray Hill, NJ.
By this time, use of B in the early Unix system was already pretty
much at an end; early C had already taken over
(see The Development of the
C Language). In fact, by September 1973, the operating system
had already been translated into C and most of the B utilities
converted.

The memos shown here were based on an earlier document,
User's Reference to B,
by Ken Thompson.
The Unix dialect of B closely followed the Honeywell version
described here--the compiler front-ends were the same,
but of course the Unix system calls were present, and the
GCOS-specific I/O stuff absent. The basic library must
have looked very similar.

Indeed, there were never very many B programs on early Unix.
The compiler itself was written in B, and a few of the utilities,
for example the first /etc/glob, which expanded wild-card
characters for the shell.
This is because no compiler to machine code for the PDP-7 or PDP-11 was
ever built; the Unix B compilers produced interpreted, threaded
code that wasn't efficient enough to write a whole system in.

On the other hand, B, with a real compiler, flourished in a modest way on the Honeywell
machines, as indicated by this CSTR.
Moreover, it had direct use and even progeny elsewhere, especially
at the University of Waterloo in Ontario.
It apparently lives on today: see
Thinkage Ltd. UW Tools Package,
for example.

A final note of possible historical interest or amusement:
so far as Brian and I can remember, the Tutorial contains the
first instance of a "Hello, world" program.

Copyright © 1997 Lucent Technologies Inc. All rights reserved.

Last fiddled 29 May 2000, adding PDF distillations of the OCR.
