---
layout: post
title:  "August bpython sprint"
date:   2014-08-31 18:35:00
alias:  /2014/08/31/august-bpython-sprint.html
tags: bpython
---

I just took a vacation for a week! I spend time with my family in Washington
State mostly doing fun outdoorsy things.
I didn't answer email for the week but I did try to get some
work in on bpython: only as much as was fun to do at the time. I made a [list of 
issues](https://github.com/bpython/bpython/issues?q=milestone%3ATom-august-sprint)[^1]
to try to tackle and chose the deadline of the first day of the next batch of
Hacker School to finish them. I only have half of
the original tasks checked off so far and still have today and tomorrow to improve that ratio,
but I wanted to do a retrospective while I had time and they were fresh on my mind.

What I got done
===============

1. [Readline kill and yank in
   bpython](https://github.com/bpython/bpython/issues/344)

    I got to learn all about the readline kill buffer for this issue. In bpython
    we don't use readline in order to process keyspresses one at a time, which
    a lot of reimplementing readline ourselves.[^2] I had to change
    the way our [readline keybindings get assigned in
    code](https://github.com/bpython/bpython/blob/master/bpython/curtsiesfrontend/manual_readline.py)
    to let readline edit functions return what was just killed.
    I'm pretty happy with how this turned out, and think it'll be useful
    in the future.
    
    It turns out multiple kills with the same readline
    kill command add to the kill buffer cumulatively! I wasn't aware of this
    feature until I bothered to read some readline docs.
    I included this in our kill / yank feature and am already
    finding it useful.

2. [Open config in bpython](https://github.com/bpython/bpython/issues/354)

    You can now open your bpython config from bpython! It's F3 by default.
    Increasing discoverablility
    of config options was the goal here, alongside the new help
    feature which shows all keybindings (F1 by default).
    Since we already had all the tools in place for launching
    an eternal editor, increasing use of the config file will hopefully be worth
    the feature creep.

3. [Readline-style incremental
   search](https://github.com/bpython/bpython/issues/355)

   This is a feature many people have asked for (though except for
   [rntz](http://www.rntz.net/) they've been anonymous folks from
   irc): ctrl-r and ctrl-s should work as they do in readline.
   Like the readline kill buffer, working on it opened
   my eyes to the legacy of readline - effective use of the command line
   is a skill useful at a scale larger than programming languages, and
   obeying UX conventions like readline enable folks experienced.
   My implementation is similar to that of bash, but does highlighting
   differently and matches all locations on a given line simultaneously, more
   like fish-style up arrow completion.

   We still have to decide who to dissapoint with the default keybindings:
   longtime bpython users who
   expect ctrl-r to be rewind and ctrl-s to be save or users that expect
   these keys to work as they do almost everywhere else. For the moment
   the incremental search functionality is on *meta-r* and *meta-s*.

4. [Repl painting
   tests](https://github.com/bpython/bpython/commit/bd11ff7e8c7a180af911d3b1d40533c15ba6c4b6)

   I've been causing a lot of regressions in our not-very-well-tested codebase
   so I'm very excited to start recording the behavior of bpython like this
   to prevent these regressions. Adding a custom diff display to
   [curtsies](https://github.com/thomasballinger/curtsies) made this pretty
   fun:

   ![diff example](http://ballingt.com/assets/diff.gif)

5. [Travis CI](https://github.com/bpython/bpython/issues/358)

   Another source of regressions for me is failing to test everything in
   all of the Python versions we support (Python 2.6, 2.7, 3.3, and 3.4).
   Running our tests in all environments is certainly possible to do locally
   (I use tox to do this) but having the extra, automatic check is nice.
   It was fun to push their terminal emulator to the limit: 

   <blockquote class="twitter-tweet" lang="en"><p>Unfortunately TravisCI
   doesn&#39;t implement blink, but they&#39;re underlined: <a
   href="https://t.co/5vF3xohp1R">https://t.co/5vF3xohp1R</a></p>&mdash;
   Thomas Ballinger (@ballingt) <a
   href="https://twitter.com/ballingt/statuses/503377586559668224">August 24,
   2014</a></blockquote>
   <script async src="//platform.twitter.com/widgets.js"
   charset="utf-8"></script>

I also fixed a few issues that the intrepid master branch-using [myint](https://github.com/myint) found,
closed an issue I'd opened as not actually useful, and cleaned up some tests.

What I didn't get done
======================

1. [Autoindent](https://github.com/bpython/bpython/issues/331) and [autocompletion](https://github.com/bpython/bpython/issues/327) for multiline commands

   Originally brought up because they would make monkeypatching in completion
   and highlighting possible in [bpython for Hy](https://github.com/thomasballinger/bphython),
   I didn't want to work on these because they would require refactoring
   existing parenthesis counting code in bpython that didn't look like fun to
   touch.

2. [Support for ASCII terminals](https://github.com/bpython/bpython/issues/295)

   Figuring out how to test this didn't seem fun, but by narrowing the scope
   of the bug from "fix encodings" to "make ascii work," I feel pretty good
   about tackling it soon.

3. [Clear screen bug](https://github.com/bpython/bpython/issues/343)

   This bug looks pretty involved, it's for a feature I don't like much
   (the temporary "Welcome to bpython" banner), and it might change completely if
   the we ever start using a [separate process](https://github.com/bpython/bpython/issues/353)
   for bpython (which isn't going to be until after the next release).
   I'll probably pair with someone on it next week: it's the kind of bug
   that I don't trust myself to understand fully without collaboratively
   building and externally communicating that udnerstanding.

4. [Help browser in bpython](https://github.com/bpython/bpython/issues/294)

   I though I knew how to go about this, but was thrown for a loop by some
   limitations I forgot terminals had with regard to saving the current
   session and putting a fullscreen window over it. A few false starts have given me
   the understanding of the problem I need to tackle it in the future.

So what's the point?
====================

There isn't much of one - I just wanted a chance to reflect on the work I had
done. I'm still cogitating on what lessons to take from this work.
I'll probably crib from this post a bit for the changelog
in the next bpython release.

It is interesting to see the fun pieces get done
and the less fun ones not, and to think
about what makes some programming tasks more enjoyable than others.

I'm very grateful to folks like myint who are running master and reporting
bugs - if you'd like to join them check out the [contributing docs](http://docs.bpython-interpreter.org/contributing.html)!

[^1]: I did add a few things to the sprint retroactively that I wanted to talk
    about here, work I did the weekend before I was on vacation.

[^2]: I [don't entirely understand how readline works](https://github.com/bpython/bpython/issues/363)
    so I'm not confident we shouldn't be using it somehow.
    We may go as far as implementing readline vi keybindings and
    using user preferences from .inputrc in the future, so it would sure be nice to harness it.

