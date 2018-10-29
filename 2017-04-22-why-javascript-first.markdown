---
date: 2017-04-22T23:00:00Z
title: Why teach JavaScript first
url: /why-javascript-first/
---

A couple of tweet threads from Sarah Drasner caught my eye today:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">The decision by Stanford to use JavaScript to teach instead of Java has me wondering, why not Python?</p>&mdash; Sarah Drasner (@sarah_edo) <a href="https://twitter.com/sarah_edo/status/855810196219846656">April 22, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">After a few hours of this tweet going and some interesting discussion in the thread, I think my opinion on this has swayed <a href="https://t.co/1z1z5EJyNf">https://t.co/1z1z5EJyNf</a></p>&mdash; Sarah Drasner (@sarah_edo) <a href="https://twitter.com/sarah_edo/status/855934231582789632">April 22, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

I agree: teach intro programming in JavaScript.
Although I value folks doing and collecting Real Research like [Mark
Guzdial](https://computinged.wordpress.com/),
[Mel Chua](http://blog.melchua.com/), and [Phillip
Guo](https://cacm.acm.org/blogs/blog-cacm/176450-python-is-now-the-most-popular-introductory-teaching-language-at-top-u-s-universities/fulltext)
on this and other computing education issues,
my opinion on this comes mostly from my own experiences teaching and learning.

Although I agreed with the conclusion, I saw a bunch of reasons I didn't agree
with in the first thread, the majority of reasons in fact!
So I thought I'd make my own argument here.

I think both Python and JavaScript are good options, and I truly cherish the
greater Python community within which I've been nurtured as a programmer for
over a decade. And any language that your friends use, or that an accessible
community of practice uses, or even is just available can be the best choice
for an individual. That might be learning to program in StarCraft's trigger language;
in Excel because your parents know that;
in BASIC because that's what's in the elementary school library even though
one has no way to run these programs, then later in TI-81 calculator BASIC;
by reading books about Java that one can barely compile because an uncle
can answer questions about it; in Logo because that's what the school computer lab
teacher knew and had books on, and gives cool visual feedback.[^yup]

The right language to teach is also the one you know, and is useful in your
department, school, field, industry, etc. This is really targeted toward
making an institutional decision not telling you about your
circumstances.

Another great Twitter thread I saw today was April Wensel gathering many
people's journeys through programming languages, teed off by this:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Your first programming language may impact the route you take through the concepts, but it never limits your future learning. 🤷🏻</p>&mdash; April Wensel (@aprilwensel) <a href="https://twitter.com/aprilwensel/status/855824635123728384">April 22, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

With those caveats out of the way, I'll also lose the "I thinks" etc. in the rest of
this post - if you're confused, check the url bar to see that this is indeed
someone's personal website, not the impartial synthesis of research.

---

If you're teaching a beginning programming class, for CS majors or otherwise,
you should teach JavaScript.

Programming is a prerequisite for the study of computer science. If
programming is not explicitly taught, it will instead be implicitly selected
for because computer science is a hell of a lot easier to learn if you can
program. The best way to learn to program to get sucked into it. That takes
a reason to care about it so motivating as to power you through the morass of
knowledge and skills you'll need to learn to make headway.

If a student has significant Excel experience and it trying to accomplish work
at the edge of its capabilities, VBA may be a better fit for them.
If they're a computational biologist and the libraries they need are Python
libraries, they should go with that. But lacking a specific need – that is, in
an introductory programming class for CS majors, not biologists learning to
program – then a need we all have it to interact with the universal program we
all use: the web browser.

You already have an IDE for JavaScript installed, and you mostly know how to use it because
you spend 6 hours a day in it already. In a few lines you can accomplish real
tasks people care about like selecting the ads on a page and hiding them, or [gender-swapping
the pronouns on a page](http://www.daniellesucher.com/2011/11/11/jailbreak-the-patriarchy-my-first-chrome-extension//).
Without installing anything, or even having to think about the idea of a
library or module, you can start playing with image processing, drawing shapes
or doing pixel-wise operations. And most importantly, in an environment with
which you're already familiar - learn Cmd-Option-I in Firefox or Chrome and
you're off to the races, with a constant reminder of this newly developing
superpower every time you visit a web page.

HTML and CSS inevitably come into this sort of JavaScript curriculum a bit,
which is fine. Skills like "copying a series of words and symbols, then
finding the one character that differs" that students will need to learn carry
over from these languages as well.

Experienced programmers overestimate the annoyance to beginners of the frustrations
they have with the language.
Beginners often don't read error messages anyway.
Beginners don't know they don't want weak typing.
Beginners don't need operator overloading.
Beginners don't care about undefined instead of KeyError.

I haven't specifically described many motivating examples in JavaScript, and
it's very important to find these - it'd be possible to port a Java textbook over
to JavaScript without taking advantage of the browser, and all you've get
would be a still-boring book, though hopefully half the size because you'd no longer
be explaining how to write software at scale anymore with static types and
getters and setters and private data - useful stuff, but not things beginners
care about. But hopefully these spring to mind: A browser extension that
tracks browsing; a bookmarklet that hides content from a website and quizzes
players on it or filters email. A web site! To many beginners programming
means smartphone apps and websites. Start there!

Finally: yes, there are more things to learn than JavaScript can teach.
But the goal of a first programming class has to be to hook the students
and to give them the power to scratch their own itches. Let the experienced
programmers test out of the class and get right to their beautiful Hindley Milner or
lucrative programming interview prep. A first programming class has to
get students programming, which takes time and fortitude, which require
caring for some reason. For those students who didn't have the head start of
a burning drive to decompile Minecraft to write plugins for their server
to impress their friends at age 8, there needs to be a door later.

---

I'd love comments on this [on
Twitter](https://twitter.com/ballingt/status/856038940389687296) and I'll hopefully add to it later.

[^yup]: Yup, I'm describing my preteen exposure to programming
