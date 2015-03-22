---
layout: post
title:  "Comparing key detection code in Blessed and Curtsies"
date:   2014-11-29 16:05:00
alias:  /2014/11/29/key-detection-code.html
---

[lmontopo](http://lmontopo.github.io/) and I recently contributed
to jquast's [wcwidth](https://github.com/jquast/wcwidth), a pure Python
implementation of the system call of the same name. Contributing was really
easy, and jquast was very supportive and helpful. Working on wcwidth led me to 
(re)discover Blessed, a terminal wrapper used in wcwidth
to assist with manually inspecting whether character sequences were the right
length in wcwidth.

---------------

[Blessings](https://github.com/erikrose/blessings) is "a thin, practical
wrapper around terminal capabilities in Python." [Blessed](https://github.com/jquast/blessed)
is jquast's fork that slightly thickens this wrapping of terminal capabilities
with useful features like keypress detection.
[Curtsies](curtsies.readthedocs.org) is a library I wrote for 
[bpython-curtsies](http://ballingt.com/2013/12/21/bpython-curtsies.html)
that probably crosses the line from wrapper to whatever something that isn't
a wrapper anymore is.
Originally I wrote my own terminal interaction code, but in
June I slotted in Blessings and it improved the code a bunch. Upon looking
into Blessed I've found I'm again duplicating functionality, and I'm excited to
make some net negative lines of code commits soon.

Both Curtsies and Blessed implement context managers[^contextmanagers] for terminal modes like cbreak[^cbreak] (and
do so much the same way) and have some capabilities for formatting strings
containing [terminal control sequences](http://en.wikipedia.org/wiki/ANSI_escape_code)
(Curtsies uses [formatted string objects](http://curtsies.readthedocs.org/en/latest/index.html#document-FmtStr)
with custom methods while Blessed uses functions that act on strings containing
a [wider
variety](https://github.com/jquast/blessed/blob/master/blessed/sequences.py) of terminal control sequences than Curtsies allows).
But I was most interested in comparing our libraries' approaches to detecting keys.

Mapping bytes read from stdin to what key a user probably pressed to cause
those bytes to be there is something [Darius](http://wry.me/~darius/)
helped me think through with [Sturm](https://github.com/darius/sturm),
though I've since strayed significantly from his clean ideas.
If you're going to follow along with all the details of this post you might open up
or pull down [Blessed](https://github.com/jquast/blessed/blob/1e22efd601f080eaa62e3bd021083b9dae7025d4/blessed/terminal.py#L537)
and [Curtsies](https://github.com/thomasballinger/curtsies/blob/d15af46a490654664e04121f6c84805e7321c243/curtsies/input.py#L144).

I prefer Blessed's approach to almost everything that Curtsies also does.
Here are some of the differences between the two and why I like the
approach Blessed takes for each. If you're looking for general takeaways,
scan the post for the bits in **bold**.

Ways Blessed's key detection is nicer than Curtsies':
=====================================================

* Calling `next()` on a `curtsies.Input` object is the equivalent to
  `blessed.Terminal.inkey()`. Because `inkey()` is a normal method, it can
  accept optional arguments, while configuring the behavior of calling `next()`
  on a `curtsies.Input` object behaves is done at instantiation.

  First general takeaway: **make the magic optional.**
  I think it's cool that `Input` objects implement the Python iterator
  protocol,[^iterator] but
  it's more difficult to understand (and to write about in a blog post).
  It's also nice to document the behavior of a method in the docs *for that
  method*, though probably fine if defaults can be set in a constructor if that makes the api
  more convenient. I should have a method like `inkey()` and then alias
  it to `__next__`.

* Blessed's `inkey()` method returns `Keystroke` objects, while calling `next()`
  on a Curtsies `Input` object returns unicode strings or `Event` objects.
  Curtsies makes up new names for these, while Keystroke
  objects include original Curses names and convenient aliases.

  Building off of the Curses names instead of making up new ones is mostly
  a matter of taste. But the potential for typos and not having the assistance
  of a linter just isn't worth the slightly prettier syntax (my original
  justification) of
  
      if key in (u'<SPACE>', u'<ESC>)`:
          do_stuff()

  The embarrasingly obvious general takeaway: **use enums or constants instead of strings.**

* Less keys are detected by Blessed than Curtsies. For the most part these
  could easily be added so this worth mentioning except
  for the decision to not to support old-style meta (which adds 128 to the value of the
  key pressed).

  This is because Blessed assumes bytes read in from stdin are always individually decodable
  with that stream's encoding, so old-style meta keys cause decoding errors.
  This allows for a nice decomposition of the problem:
  first convert read bytes to unicode characters, then detect
  multiple characters which are part of a terminal control sequence.

  The lesson for me here: **resist implementing features that are hard!**
  Next time I get that urge to implement something for completion, I should put it in
  a branch for that hack session - not everything has to end up in master.
  [doy](http://tozt.net/) also advised me not to worry about meta keys - they're
  terrible, why would you want to support them?
  The decomposition Blessings uses of bytes to unicode
  and unicode to key sequence really cleans things up, and regardless of how
  correct it is (though it sounds like it's pretty correct) it's worth
  considering an assumption that makes writing the code so much easier.

* Blessed's `inkey()` is really just for keys, while Curtsies' `next(Input)`
  might return a SigIntEvent or a PasteEvent, and
  custom events can be scheduled as well. The goal of the Input object in
  Curtsies was to write code like:

      for event in curtsies.Input():
          # react to event

  with OS events included in this loop.
  
  It wouldn't be hard
  to build a reactor object using Blessed's `inkey()` since signals[^signals] can
  interrupt it. The Curtsies `Input` object can be interrupted by a signal from
  another thread - but that would be possible to simulate by sending a signal
  that would interrupt the call. The Blessings decomposition is nicer than the one
  Curtsies uses, with the `Input` responsible for being a reactor for all
  events.

  Blessed also generalizes the idea of signal interruption, while I considered each
  signal separately as I wanted to write a handler for it in Bpython.
  When `curtsies.Input.__next__` is interrupted it can return
  the signal as an event for SigInts only.

  This decision was wrapped up in other concerns as well at the time,
  but the lesson I take from it is to **more aggresively modularize code.**
  As my needs for the `Input` object changed I should have reconsidered its
  design.

* Blessed carefully uses terminal capabilities to build key sequences, and
  then augments them with empirically found sequences.
  Curtsies just looks for key sequences that worked for me or that Bpython
  users suggested. When I found some keys weren't detected by curses, I
  abandoned its key names altogether, but **building onto existing standards**
  in a compatible way is a better idea.

* The Blessed code is just better.

  jquast's comments
  assume the reader knows Python, and that they don't know the ins and
  outs of terminals. They're great comments!
  **Write more comments about domain-specific concerns!**

  Another code quality difference: I should **use existing idioms**
  in places I made up my own thing.
  Even if it's not a performance bottleneck,
  if there's a standard library solution to a problem I ought to use it
  because it better communicates the problem being solved.
  Ways Blessed does this better than Curtsies:

  * using stdlib incremental decoder `encodings.utf_8.IncrementalDecoder`
    instances where I use a state machine I build myself[^excuse]
  * using a `collections.deque` for read bytes buffer - sure inserting at the
    beginning of a 6 item `list` isn't a big deal performance-wise, but `deque`
    also describes how we're going to use it
  * using existing infrastructure to discover key sequences (curses)

  I like that Blessed uses methods where I use local functions.
  Besides being more testable, it's clearer to see what parameters
  something takes, particularly since instance variable references
  are pretty obviously distinguishable from local variable references in Python,
  while outer scope references aren't.

* Both libraries process one additional byte at a time, but `inkey()` always reads a single
  byte at time with `os.read(stdin.fileno(), 1)` while Curtsies tries to read
  as much as possible and utilizes this information about what came in on the
  same read - for example, to tell the difference between plain escape key
  and escape-prepended other key.
  I think my approach is more elegant, but it doesn't work.[^badidea]
  I don't like having to do a timed delay to detect the escape key, but it
  sounds like it's necessary. **Getting it right is important.** I'll go easy
  on myself here because I only recently found out this doesn't quite work in
  the general case, and haven't observed it not working yet - but if I were
  going to use it perhaps it was my responsibility to research whether
  it worked generally.
  
  Never reading a byte that isn't part of a requested keypress also
  allows a Blessed `Terminal` object to share stdin with other readers,
  while the Curtsies `Input`
  object should have complete control since it might read too many
  bytes.[^ungetc]

Ways Blessed's key detection is different from Curtsies':
=========================================================

* Paste events - Curtsies tries to detect that bytes read are probably due to
  the user pasting text because they were entered very quickly by setting a
  flag when more than ~10 bytes are read on a single
  `os.read(sys.stdin.fileno(), 1024)`. I really like
  this feature, but it could be approximated by timing when keypresses happen
  without modifying Blessed's code at all.

* Blessed's `inkey()` requires the developer to have
  manually set cbreak or raw mode.
  Curtsies enters cbreak itself and optionally installs a signal handler
  that can cause an event signalling that the SIGINT occurred from the call.
  I wanted the library user not to have to worry about this (and this
  difference reflects the the difference between a terminal wrapper and
  a library that provides a service more separated from implementation),
  so I think this difference was justified.
  I think I'll keep this functionality in the event reactor object I hope
  to write for Curtsies that will use Blessed's `inkey()`.

I hope to leverage Blessed for key detection and remove the analogous bits of the Curtsies code.
Not only does this the make sense from an API perspective,
I think the Blessed code is a lot better. In order to do this, I'll want to:

* add escape key-prepended sequences to Blessed
* add support in Blessings for currently supported features in Curtsies
  like distinguishing ctrl-left and ctrl-right from normal forms
* add cursor position querying to Blessed - right now Curtsies
  [`Window` objects](http://curtsies.readthedocs.org/en/latest/index.html#document-window)
  are coupled with
  [`Input` objects](http://curtsies.readthedocs.org/en/latest/index.html#document-Input)
  to make this work properly
* build a reactor/scheduler object with the tools Blessed
  provides.


[^badidea]: Darius says on his new computer
            this doesn't work anymore - the escape byte gets written first in such a way
            that the `os.read` returns it alone, and then a later read
            gets the rest of the signal.
            So it sounds like the delay for escape key is probably required

[^contextmanagers]: Ever used `with open('file.txt')` in Python? That's a
                    [context
                    manager](https://docs.python.org/2/reference/datamodel.html#context-managers), and files (which ought to be closed no
                    matter what) and setting terminal modes (which ought to
                    set back to what they were when the program exits so the
                    cursor is visible again etc.) are good candidates for
                    context managers - there's cleanup we want to do whether
                    we leave a block of code normally or by a raise exception.
                    It does the same thing `try: ... finally: ...` does,
                    so check those out first if you're confused.

[^cbreak]: You know how `raw_input` doesn't return with what the user typed
           until they hit enter? That's because the tty driver buffers
           typed characters. (see [man
           cbreak](http://linux.die.net/man/3/cbreak)) But you can turn that
           off with things like `cbreak` so raw_input() will return a single
           key pressed.

[^iterator]: The [Python iterator
             protocol](https://docs.python.org/2/library/stdtypes.html#iterator-types) allows
             an object to be used in a for loop. The important bit here is the
             `.__next__()` method (`.next()` in Python 2) which will get
             invoked each cycle through the for loop to get the value to
             assign to the target variable before the body of the for loop is
             executed.

[^excuse]: To give myself a break on this one, this was made necessary by
           the decision to detect old-style meta key combinations,
           but at least I could have mimicked the interface in constructing
           my own incremental decoder.

[^ungetc]: But perhaps this would have been ok with `ungetc`? I'd appreciate advice on
           whether `ungetc` is reasonable to use on stdin and how to nicely use
           it from Python because it could be useful for cursor query code I
           want to write for Blessed.

[^signals]: [Signals](http://en.wikipedia.org/wiki/Unix_signal) like
            [SIGINT](http://en.wikipedia.org/wiki/Unix_signal#SIGINT)
            (which is triggered by ctrl-c) are messages from the operating
            system to the running program which in Python can interrupt normal
            execution of the program and trigger execution of other code
            called a signal handler. They can interrupt blocking system calls
            like `os.read(sys.stdin.fileno())`.
