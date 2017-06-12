---
aliases:
    - /2014/05/13/bpython-progress-update.html
    - /2014/05/13/bpython-curtsies-release.html
date: 2014-05-13T09:20:17Z
tags: bpython
title: bpython curtsies is live
url: /bpython-curtsies-release/
---

bpython 0.13 has [just been released](https://groups.google.com/forum/#!topic/bpython/jtvr4l1Snkc)
and it includes bpython-curtsies, the bpython
frontend I've been working on on and off for 9 months. If you're
not reading on your phone or in bed with an ipad, take a moment to try it out:

    $ pip install bpython[curtsies]
    ...
    $ bpython-curtsies
    >>> s = 'abc'
    >>> s.   <look at that great autocompletion and call a method>
    >>> <ctrl-R to undo that last command>
    >>> <ctrl-D to exit bpython>
    $ echo "you're back in the shell!"

![bpython-curtsies scroll demo](/assets/bpython-curtsies-scroll-demo-large.gif)

Also check out features like F7 to edit and reevaluate the current session,
dictionary key autocompletion, and ctrl-X to edit the current line.
([Read the full changelog](https://groups.google.com/forum/#!topic/bpython/jtvr4l1Snkc))

What's next? Hopefully a bunch of bugfixes! Report those bugs or feature
requests at [the repo page](https://bitbucket.org/bobf/bpython/issues). Maybe
make some [more gifs](http://ballingt.com/2013/12/21/bpython-curtsies.html).

More long term, I'd like to factor shared code out of bpython.repl.Repl
to make it more reusable - first up is completion logic.
Maja did an [excellent job](https://bitbucket.org/bobf/bpython/pull-request/41/autocomplete-for-dictionary-keys-fixes-226/diff)
adding dictionary key completion, but
it was much harder than it would have been if the completion were less
monolithic.

I wish I'd gone with an evented architecture instead of the "simpler"
system I used. Initially it really was simple: block on reading
a character from stdin, then react to it. But paste events, signal
handling, and needing to jump back and forth between the UI loop
and executing code all made this more complicated. Using greenlets
to have multiple Python call stacks
was a cool way to make this manageable (and
[was cleaner than threads](https://bitbucket.org/bobf/bpython/commits/989d8880cecbd0a10be99be516bb6fd73bc2fa23)),
but something event-driven would have been simpler and more
modular. Maybe I'll look at fixing this.

I want to investigate getting curtsies-like functionality into urwid.
Ian Ward, the creator of [urwid](http://urwid.org/), has done some work
on getting this kind of command line interface functionality in urwid.
I'd love to help with that.

When I started writing a new bpython frontend, I jusified the effort with the thought
that the startup time of bpython-urwid was too bad for it to be fun for me
to use. Writing [my own terminal
wrapper](https://github.com/thomasballinger/curtsies) was really fun, but it was also
the lazy way out - I should have learned to integrate it into urwid.
Maybe I could speed up urwid! Or maybe I don't care about that startup cost anymore.
I learned a lot building my own thing, but using urwid is probably the Right Thing
to do.

I really want to help with [bipython](https://github.com/ivanov/bipython),
but in the short term I'll work
on abstracting code out of bpython.repl.Repl, a task that should be useful
to bpython-curtsies, bpython-urwid, normal bpython and bipython.

Thanks so much to Simon de Vlieger, Bob Farrell, Sebastian Ramacher and others
for making bpython a great project to work on. Thanks to
[my employer](https://www.hackerschool.com/) for paying me to become a better
programmer and contribute and help others contribute to open source software.
