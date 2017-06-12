---
aliases:
    - /2014/02/16/code-talk-prep.html
    - /2014/02/16/code-talk-prep
date: 2014-02-16T11:12:17Z
tags:
    - hacker-school
    - python
    - bpython
title: Prepping for a Python talk with code snippets
url: /code-talk-prep/
---

After not making much progress on bpython for a while, someone in #bpython
asked for better paste support, and I found I needed the same feature.
Building something someone else needs and building something I want are two
of my favorite things!

I'd like to give a presentation this week at Hacker School about features of
Python folks might not have seen, in the form of short, standalone code
snippets we could try to predict the output of together. (part [Gary Bernhardt's
Wat](https://www.destroyallsoftware.com/talks/wat), part [Coding
Bat](http://codingbat.com/), part
http://rubykoans.com/) I'd like to be quickly jumping back and forth between a
REPL and my examples. I considered both IPython notebook and what I think of as an emacs-style
REPL (an editor in one vertical split and REPL in the other), but settled on
good old bpython, which still just feels right to me for interactive exploration
of Python.

I previously added a send-to-editor key to [my bpython fork](https://github.com/thomasballinger/bpython/tree/curtsies), but to 
quickly get my examples into a REPL, I wanted better paste support and the
ability to automatically paste the contents of a file in: now
`bpython-curtsies --type example.py`. I've
got a folder of small example files I can run individually or examine in more
detail in the REPL.

The content is going to be all over the place, with the general pattern of
"Do you understand this code? You should, it's awesome! (see this reference if
you don't)". It will have examples as simple as "functions are first class - look at me storing
them in a datastructure," as well as more subtle things like how hoisting works in Python:

    a = 1
    def foo():
        print a
        a = 2

    foo() #UnboundLocalError: local variable 'a' referenced before assignment

I have tried to cut Python language topics roughly in half, so won't touch in
this session on object orientation, classes, attribute lookup, dunder methods,
or modules. If this session is useful to folks I'll do those in another
one covering those.

I aim to write up some notes soon, because otherwise people might take
notes when I recommend references, and I'd prefer we all spend the time playing
with a REPL :)

Hopefully the session gives people language to use to describe issues in
their code, reminds them that better techniques may well exist, and advertises
references I've found particularly useful like "Python Essential Reference",
[Dive into Python](http://www.diveintopython.net/), and
[Loop like a native](http://nedbatchelder.com/text/iter.html). When anyone asks
me how to learn Python most quickly, my answer is still to write a lot of code
and get feedback on it, but I hope a few pinprick glimpses of cool
language features will be useful too.
