---
layout: post
title:  "Writing Curtsies Docs"
date:   2014-11-22 15:30:00
alias:  /2014/11/22/writing-curtsies-docs.html
tags: temrinal
---

I finally bit the bullet and wrote [documentation for
Curtsies](http://curtsies.readthedocs.org/en/latest/) this last weekend.
This required getting familiar with a few new tools like reStructuredText
and Sphinx's additions to it, including learning enough about those projects'
internals to write a hacky
[custom directive and parser](https://github.com/thomasballinger/curtsies/blob/master/docs/terminal_output.py)
for displaying ANSI escape code-containing terminal output. But the most
interesting part was the thinking about my library's public API this forced me to do.

* The large examples are mostly about using all the components of my library
  together. Is this bad? Will it be clear that they can be used
  separately? Does this indicate these components are more coupled that I
  thought?

* Could I have written this documentation first, and then implemented the
  library?

I couldn't resist making some of the changes as I was writing the docs
so I wouldn't have to carefully describe the convoluted current state of the
project, but I tried to save all the really breaking changes for after I finished
my documentation adventures. Here's what I want to change soon:

* rename most of my objects
  * FmtStr -> ColoredString? AttributedString? FormattedString?
  * FSArray -> Grid? ColoredGrid?
  * Input.scheduled_event_trigger is a terrible name - it's a method on an
    Input object that returns a callback which when called schedules an event
    for at a specifid future time. I'm not sure what to call it instead.

* less magic, or make the magic optional; there should be regular methods to
  which `__setitem__` and `__setitem__` defer

* FmtStr objects should index on characters, but FSArrays should be
  [width-indexed](http://curtsies.readthedocs.org/en/latest/index.html#fmtstr-len-vs-width)

* use the terminal width of strings isntead of their length for formatting

* use constants instead of strings for key names, ideally copy someone else's
  names; I resisted this because I like the way comparing a retrieved event
  against strings looked, but it's just not worth the confusion

* normalize key names - eliminate inconsistencies like `<Ctrl-a>` but
  `<Esc+Ctrl-A>`

* some special characters lose their special names when used with modifier keys, for example:
  `` <TAB>, <Shift-TAB>, <Esc+Ctrl-Y>``

* the constructor for FmtStr should be useful, the current `__init__` that
  requires passing instances of a class that isn't part of the public API
  so some internal code is slightly prettier isn't worth it

* `Window.keep_last_line` is the wrong name, it should be `Window.keep_cursor_line`

* I shouldn't expose the `fileno` method on Input because it's hard to use
  properly

* the coroutine-imitating interface of Input isn't useful; either I should come up
  with an example of how Input could be used like a coroutine or it shouldn't be described
  it as such

Other lessons from doc writing:

* [Mary](http://maryrosecook.com/) suggested docs that are all on a single page are nice,
  and it turns out you can still use the provided ReadTheDocs
  build process and theme with the single page HTML sphinx build target.

* It's fun to write things the will show up on a pretty website where anyone
  can read them, and it's most fun to write documentation when you're procrastinating
  something else. (I should have been catching up on email and practicing a
  talk last weekend.)
