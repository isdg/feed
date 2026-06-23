---
title: "Reflections on Trusting Trust"
url: https://www.win.tue.nl/~aeb/linux/hh/thompson/trust.html
published: "1984-08-01T00:00:00Z"
feed: thompson
guid: https://www.win.tue.nl/~aeb/linux/hh/thompson/trust.html
---

# Reflections on Trusting Trust

Ken Thompson's "cc hack" - Presented in the journal, Communication
of the ACM, Vol. 27, No. 8, August 1984, in a paper entitled
"Reflections on Trusting Trust", Ken Thompson, co-author of UNIX,
recounted a story of how he created a version of the C compiler
that, when presented with the source code for the "login" program,
would automatically compile in a backdoor to allow him entry to
the system. This is only half the story, though. In order to hide
this trojan horse, Ken also added to this version of "cc" the
ability to recognize if it was recompiling itself to make sure
that the newly compiled C compiler contained both the "login"
backdoor, and the code to insert both trojans into a newly
compiled C compiler. In this way, the source code for the C
compiler would never show that these trojans existed.

Reflections on Trusting Trust

by Ken Thompson

Introduction

I thank the ACM for this award. I can't help but feel that I am
receiving this honor for timing and serendipity as much as
technical merit. UNIX swept into popularity with an industry-wide
change from central main frames to autonomous minis. I suspect
that Daniel Bobrow (1) would be here instead of me if he could not
afford a PDP-10 and ad had to "settle" for a PDP-11. Moreover, the
current state of UNIX is the result of the labors of a large
number of people.

There is an old adage, "Dance with the one that brought you,"
which means that I should talk about UNIX. I have not worked on
mainstream UNIX in many years, yet I continue to get undeserved
credit for the work of others. Therefore, I am not going to talk
about UNIX, but I want to thank everyone who has contributed.

That brings me to Dennis Ritchie. Our collaboration has been a
thing of beauty. In the ten years that we have worked together, I
can recall only one case of miscoordination of work. On that
occasion, I discovered that we both had written the same 20-line
assembly language program. I compared the sources and was
astounded to find that they matched character-for-character. The
result of our work together has been far greater than the work
that we each contributed.

I am a programmer. On my 1040 form, that is what I put down as my
occupation. As a programmer, I write programs. I would like to
present to you the cutest program I ever wrote. I will do this in
three stages and try to bring it together at the end.

Stage I

In college, before video games, we would amuse ourselves by posing
programming exercises. One of the favorites was to write the
shortest self-reproducing program. Since this is an exercise
divorced from reality, the usual vehicle was FORTRAN. Actually,
FORTRAN was the language of choice for the same reason that
three-legged races are popular.

More precisely stated, the problem is to write a source program
that, when compiled and executed, will produce as output an exact
copy of its source. If you have never done this, I urge you to try
it on your own. The discovery of how to do it is a revelation that
far surpasses any benefit obtained by being told how to do it. The
part about "shortest" was just an incentive to demonstrate skill
and determine a winner.

**Figure 1.** A C program that prints its own source — a
self-reproducing program (quine). The array `s` encodes the body
of the program from the `'0'` line to the end; `main` prints the
declaration of `s`, then the bytes of `s` as data, then `s` again
as text. It shows a program can be given knowledge of itself.

```c
char  s[] = {
    '\t',
    '0',
    '\n',
    '}',
    ';',
    '\n',
    '\n',
    '/',
    '*',
    '\n',
    /* ... 213 lines deleted ... */
    0
};

/*
 * The string s is a
 * representation of the body
 * of this program from '0'
 * to the end.
 */

main()
{
    int i;

    printf("char\ts[] = {\n");
    for(i = 0; s[i]; i++)
        printf("\t%d, \n", s[i]);
    printf("%s", s);
}
```

Figure 1 shows a self-reproducing program in the C programming
language. (The purist will note that the program is not precisely
a self-reproducing program, but will produce a self-reproducing
program.) This entry is much too large to win a prize, but it
demonstrates the technique and has two important properties that I
need to complete my story: (I) This program can be easily written
by another program. (2) This program can contain an arbitrary
amount of excess baggage that will be reproduced along with the
main algorithm. In the example, even the comment is reproduced.

Stage II

The C compiler is written in C. What I am about to describe is one
of many "chicken and egg" problems that arise when compilers are
written in their own language. In this ease, I will use a specific
example from the C compiler.

C allows a string construct to specify an initialized character
array. The individual characters in the string can be escaped to
represent unprintable characters. For example,

"Hello world\n"

represents a string with the character "\n," representing the new
line character.

**Figure 2.** Idealized code from the C compiler that interprets a
character escape sequence: read a char; if it is not a backslash,
return it; otherwise read the next char and map `\\`->backslash,
`n`->newline. Note it returns `'\n'` — the compiler must _already_
know what newline is.

```c
    ...
    c = next();
    if(c != '\\')
        return(c);
    c = next();
    if(c == '\\')
        return('\\');
    if(c == 'n')
        return('\n');
    ...
```

Figure 2 is an idealization of the code in the C compiler that
interprets the character escape sequence. This is an amazing piece
of code. It "knows" in a completely portable way what character
code is compiled for a new line in any character set. The act of
knowing then allows it to recompile itself, thus perpetuating the
knowledge.

**Figure 3.** The same routine extended to add `"\v"` (vertical
tab), returning the character constant `'\v'`. Recompiling fails:
the running compiler binary does not yet know `\v`, so `'\v'` is
not legal C — a bootstrap problem.

```c
    ...
    c = next();
    if(c != '\\')
        return(c);
    c = next();
    if(c == '\\')
        return('\\');
    if(c == 'n')
        return('\n');
    if(c == 'v')
        return('\v');
    ...
```

Suppose we wish to alter the C compiler to include the sequence
"\v" to represent the vertical tab character. The extension to
Figure 2 is obvious and is presented in Figure 3. We then
recompile the C compiler, but we get a diagnostic. Obviously,
since the binary version of the compiler does not know about "\v,"
the source is not legal C. We must "train" the compiler. After it
"knows" what "\v" means, then our new change will become legal C.
We look up on an ASCII chart that a vertical tab is decimal 11. We
alter our source to look like Figure 4. Now the old compiler
accepts the new source. We install the resulting binary as the new
official C compiler and now we can write the portable version the
way we had it in Figure 3.

**Figure 4.** The fix: temporarily return the literal numeric
value `11` for `\v`. Once this binary is built and installed, the
compiler knows `\v`, so the source can be restored to the
self-referential `'\v'` of Figure 3 and it compiles. The knowledge
now lives only in the binary, not the source.

```c
    ...
    c = next();
    if(c != '\\')
        return(c);
    c = next();
    if(c == '\\')
        return('\\');
    if(c == 'n')
        return('\n');
    if(c == 'v')
        return(11);
    ...
```

This is a deep concept. It is as close to a "learning" program as
I have seen. You simply tell it once, then you can use this
self-referencing definition.

Stage III

**Figure 5.** Skeleton of the compiler's top-level control:
`compile(s)` is handed a line of source to compile.

```c
compile(s)
char *s;
{
    ...
}
```

Again, in the C compiler, Figure 5 represents the high-level
control of the C compiler where the routine "compile" is called to
compile the next line of source. Figure 6 shows a simple
modification to the compiler that will deliberately miscompile
source whenever a particular pattern is matched. If this were not
deliberate, it would be called a compiler "bug." Since it is
deliberate, it should be called a "Trojan horse."

**Figure 6.** A Trojan horse. When the source being compiled
matches a pattern (e.g. the UNIX `login` command), the compiler
deliberately miscompiles it to insert a bug (e.g. a backdoor
password). The `login` source itself stays clean.

```c
compile(s)
char *s;
{
    if(match(s, "pattern")) {
        compile("bug");
        return;
    }
    ...
}
```

The actual bug I planted in the compiler would match code in the
UNIX "login" command. The replacement code would miscompile the
login command so that it would accept either the intended
encrypted password or a particular known password. Thus if this
code were installed in binary and the binary were used to compile
the login command, I could log into that system as any user.

Such blatant code would not go undetected for long. Even the most
casual perusal of the source of the C compiler would raise
suspicions.

**Figure 7.** Two Trojan horses. `pattern1` is the `login`
backdoor (`bug1`). `pattern2` matches the C compiler itself and
inserts `bug2` — a self-reproducing routine that re-installs
_both_ Trojans into every future compiler binary. After this
compiler is installed, the source of both the compiler and `login`
is clean; the attack survives only in the binary and propagates
itself. Removing the bug from the source no longer removes the
bug.

```c
compile(s)
char *s;
{
    if(match(s, "pattern1")) {
        compile("bug1");
        return;
    }
    if(match(s, "pattern2")) {
        compile("bug2");
        return;
    }
    ...
}
```

The final step is represented in Figure 7. This simply adds a
second Trojan horse to the one that already exists. The second
pattern is aimed at the C compiler. The replacement code is a
Stage I self-reproducing program that inserts both Trojan horses
into the compiler. This requires a learning phase as in the Stage
II example. First we compile the modified source with the normal C
compiler to produce a bugged binary. We install this binary as the
official C. We can now remove the bugs from the source of the
compiler and the new binary will reinsert the bugs whenever it is
compiled. Of course, the login command will remain bugged with no
trace in source anywhere.

Moral

The moral is obvious. You can't trust code that you did not
totally create yourself. (Especially code from companies that
employ people like me.) No amount of source-level verification or
scrutiny will protect you from using untrusted code. In
demonstrating the possibility of this kind of attack, I picked on
the C compiler. I could have picked on any program-handling
program such as an assembler, a loader, or even hardware
microcode. As the level of program gets lower, these bugs will be
harder and harder to detect. A well installed microcode bug will
be almost impossible to detect.

After trying to convince you that I cannot be trusted, I wish to
moralize. I would like to criticize the press in its handling of
the "hackers," the 414 gang, the Dalton gang, etc. The acts
performed by these kids are vandalism at best and probably
trespass and theft at worst. It is only the inadequacy of the
criminal code that saves the hackers from very serious
prosecution. The companies that are vulnerable to this activity
(and most large companies are very vulnerable) are pressing hard
to update the criminal code. Unauthorized access to computer
systems is already a serious crime in a few states and is
currently being addressed in many more state legislatures as well
as Congress.

There is an explosive situation brewing. On the one hand, the
press, television, and movies make heroes of vandals by calling
them whiz kids. On the other hand, the acts performed by these
kids will soon be punishable by years in prison.

I have watched kids testifying before Congress. It is clear that
they are completely unaware of the seriousness of their acts.
There is obviously a cultural gap. The act of breaking into a
computer system has to have the same social stigma as breaking
into a neighbor's house. It should not matter that the neighbor's
door is unlocked. The press must learn that misguided use of a
computer is no more amazing than drunk driving of an automobile.

Acknowledgment

I first read of the possibility of such a Trojan horse in an Air
Force critique (4) of the security of an early implementation of
Multics. I cannot find a more specific reference to this document.
I would appreciate it if anyone who can supply this reference
would let me know.

References

Bobrow, D.G., Burchfiel, J.D., Murphy, D.L., and Tomlinson, R.S.
TENEX, a paged time-sharing system for the PDP-IO. Commun. ACM 15,
3 (Mar. 1972), 135-143. Kernighan, B.W., and Ritchie, D.M. The C
Programming Language. Prentice-Hall, Englewood Cliffs, N.J., 1978.
Ritchie, D.M., and Thompson, K. The UNIX time-sharing system.
Commun. ACM 17, 7(July 1974), 365-375. 4. Unknown Air Force
Document.
