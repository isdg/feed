---
title: "A Retrospective"
url: https://cm-bell-labs.github.io/who/dmr/retroindex.html
published: "2000-01-01T00:00:00Z"
feed: ritchie
guid: https://cm-bell-labs.github.io/who/dmr/retroindex.html
---

# A Retrospective

An Early Retrospective

Dennis Ritchie

This is a rendering of a paper from the somewhat fabled Bell System Technical
Journal, Volume 57, Number 6, Part 2, July-August 1978.
It's no longer possible to get an original copy of this edition
of the journal either
from AT&T or from Lucent; a reprinted edition,
UNIX System: Readings and Applications, Prentice-Hall, 1987
in two volumes (ISBN 0-13-938532-0 and 0-13-939845-7) is likewise out
of print.

This paper is amusing today, if only for the fact that I was already
writing retrospectives about Unix less than 10 years
after Unix appeared. When the paper was published, our own group was using
a PDP-11/70, and were well launched on the Interdata 8/32 work demonstrating
system portability, but the version for the VAX machine, which would
turn into the 32V distribution that in turn flowered into the BSD family,
wasn't ready for print. The longer retrospective today would be
very much longer.

It's also fun to remember for me because the original version was,
as documented in a footnote, written for and presented to
the Hawaii International Conference on System Sciences, Honolulu, 1977.
This was the first far-away and exotic business trip I took--driving
up to Yorktown Heights for the original Unix paper with Ken, plus a visit or so
to Bell Labs at Indian Hill near Chicago don't count as exotic--and
I was acutely aware at the time that a conference in Hawaii might
look like a boondoggle, which of course it was.

The paper itself does contain a few notable tidbits; it's a nice
snapshot, and is reasonably honest. The disk performance
analysis is naive, but the paper is good in seeing that the lack of
IPC mechanisms other than anonymous pipes (wonderful as they are)
is a serious problem that
caused endless proliferation of attempted solutions.
Much of its advice for the future (whether through Unix's influence
or otherwise) seems to have happened: hierarchical file systems,
use of at least moderately portable languages.
Some has not: "The greatest care should be taken to ensure
that there is only one format for files."
Then I was worried about card-images vs. text, now I am
harried by MS Word documents that cannot be read by MS Word.

The paper is available in these forms:

- Browsable

- PostScript

- PDF

Copyright © 2000 Lucent Technologies Inc. All rights reserved.

Last fiddled 20 October 2000.
