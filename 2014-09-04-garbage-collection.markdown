---
aliases:
    - /2014/09/21/garbage-collection.html
    - /2014/09/21/garbage-collection
date: 2014-09-04T23:15:00Z
title: Garbage collection
url: /garbage-collection/
---

Guy Steele is a smart man who gave a talk sixteen[^1] years ago about the
design[^2] of the Java[^Java] programming[^programming] language.[^language][^programming_language]
In Guy Steele's [talk](https://www.youtube.com/watch?v=_ahvzDzKdB0) "[Growing a
Language](http://www.cs.virginia.edu/~evans/cs655/readings/steele.pdf)" he
does not tell what *garbage collection* means, and says that is a task
for you to try in
your spare time. I have some spare time this day, and so will try to tell
what *garbage collection* is.

A *person* is the thing that more than one of in a group can be called folks,
but only one of cannot be called folks.

*Garbage* is trash - it is that for which there is no use. What is garbage to some
folks may not be to other folks. What is garbage in one place may not be garbage
in a place other than that first place. Then the thing which makes garbage is not what a thing is, but
if any person has use for it. The finding of what is garbage is the task of
finding if any person has use for that thing.

To *refer* to something is to point at it. I referred[^ed] to the talk of Guy Steele
in place of writing out the text of the talk.
A *reference* is a thing which only refers to an other[^other] thing.

When no one has use of this talk of Guy Steele, and no web page like this one
refers to it, it will be garbage. This will not be true for a long time.

In a programming language, things can make use of other things much like a
person makes use of things, and like the way this page refers to the talk of Guy Steele.
Things may refer to other things, in which case they have use for them.

*Collection* is the act of putting some things so they are close to each other.

Garbage collection is the act of collecting garbage, and also putting it out
of the way to make room for new things. If this task is the job of a programmer[^programmer]
in a programming language, we say the programming language does not have
garbage collection. If the programming language does this and the
programmer does not have to, we say it does have garbage collection.

A simple way to find things that are garbage might be to take a thing and
check with each other thing to see if it has use of it - to see if it refers
to it. If not, that thing is garbage. This is a method of garbage collection.

------------------------

If a left shoe has use for a right shoe, and that right shoe has use for that
left shoe, but no other things have use for one shoe nor the other, we say
they are both garbage. But this means that asking every other thing if it refers
to a first thing does not work for finding garbage for collection.

If we draw lines that go one way from each thing to all the things it refers
to, we end up with a web of such links. If this mess were
brought up in the air and shook with weak force such that groups of things
with no links to other groups fell down, each such group of things we would call a
*cycle*. Finding each cycle is a job for garbage collection, as only one of
these cycles is not garbage.

-------------------------

As well as using primitives[^primitives] I hope you all know in this text, I have also used
words Guy Steele defined[^defined] in his talk, which I point out when I do so
with small notes at the foot of the page. He may assume other primitives I
don't define to define these words; you should [read his talk](http://www.cs.virginia.edu/~evans/cs655/readings/steele.pdf) to learn those
words. I change some of his words that tell what words mean to tell what other words mean as well, so that words he uses
to define words are defined in place, but I have not done this with all the
words he uses. When I do this I put a "(" glyph to the left of these words and
a ")" glyph to the right of them. When I define words in the body of
the text, they are not words Guy Steele had need to define, or I want to
define them differently.[^exception]

[^Java]: Java is a brand name for a *computer* *programming language*

[^1]: Sixteen is twice eight

[^2]: A design is a plan for how to build a thing. To design is to build a thing in one’s
    mind but not yet in the real world — or, *better* (more good) yet, to plan how the real thing can
    be built.

[^primitives]: A primitive is a word for which we can take it for granted that we all
    know what it means. As a rule, we can speak of two or more of a thing if we
    add an “s” or “z” sound to the end of a word that names it.)


[^defined]: To define a word is to tell what it means. When you define a word, you add it to your
    *vocabulary* (a set of words)
    and to the vocabulary of each person who hears you. Then you can use the
    word. 
    
[^other]: The word other means “not the same.” The phrase other than means
    “not the same as.”

[^computer]: A computer is a *machine* (a thing that can do a task with no help,
    or not much help, from a *person*) that can do at least what *the two number
    machine* can do—
    and we have good cause to think that if a computer task can be done at all,
    then the
    two number machine can do it, too, if you put numbers in and read them out in
    the right
    way. In some sense, all computers are the same; we know this thanks to the
    work of such
    persons as *Alan Turing* and *Alonzo Church*.

[^example]: An example is some one thing, out of a set of things, that I put
    in front of you so that you can see how some part of that thing is in
    fact a part of each thing in the set.

[^programming_language]: A programming language is a language that we can use
    to tell a computer a program
    to do.

[^programmer]: One more rule for making words: if we add the *syllable* (a bit of
    sound that a mouth and tongue can say all at one time, more or
    less, in a smooth way; each word is made up of one or more syllables) “er” to a verb stem, we make
    a noun that names a person or thing that does what the verb says to do. For example, a
    buyer is one who buys. A user is one who does use.

[^programming]: To program is to make up a list of things to do and choices to make, to be done by a
    computer. Such a list is called a program. A noun can be made from a verb in such a way that it means that which
    is done as meant by that verb; to make such a noun, we add “ing” to the verb stem.
    Thus we can speak of “programming.”

[^language]: 
    A language is a *vocabulary* (a set of words) and rules for what a string of words might mean to
    a person
    or a machine that hears them.

[^ed]: We can use a verb in the past tense if we add a “d” or “ed” sound at
    the end of the verb.
    In the same way we can form what we call a past participle, which is
    a form of the
    verb that says of a noun that what the verb means has been done to it.

[^exception]: I took *Steele's* *challenge* to be *explaining* garbage
    collection *given* all the words he had defined so far at the point
    in his talk when he makes the *challenge*. *Therefore* these *footnotes*
    are only for your *convenience* and *amusement* if you haven't read the
    talk, and are all but this one *taken* *verbatim* from there.

