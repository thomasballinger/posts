---
layout: post
title:  "Python Prompt Toolkit impressions"
date:   2014-09-30 22:00:00
---

Last night I discovered Jonathan Slenders' [Python Terminal Toolkit](https://github.com/jonathanslenders/python-prompt-toolkit): a fancy interface to
the Python interpreter very much like
[bpython-curtsies](http://ballingt.com/2013/12/21/bpython-curtsies.html), the project I've
been working on on and off for a year. I was simultaneously very excited to 
see this work and a little depressed - on first glance many aspects
of the project seem better than bpython, and appears to include a terminal
wrapper at least as nice as my [curtsies](https://github.com/thomasballinger/curtsies)
library. I think there's potential for code
reuse though, and it would be really fun to talk to Jonathan about the project.
If the toolkit turns out to be flexible enough and we can add the necessary
features, it might make for a great frontend for bpython.

Jonathan says Python Prompt Toolkit isn't done, and I don't mean this
as a final product review. I just wanted to get out my reaction to it right
now. I haven't even looked at the code yet, so I'm just talking about functionality.
These are my raw, unedited thoughts mostly so I can take stock of them.

Things I love
-------------

* vi and emacs readline keybindings - 
  tos9 wants to add vi keybindings to bpython, but what Jonathan is written
  is more powerful than I had imagined.
* Multiline command editing done well!
* Doesn't execute on syntax error
* History preservation + fancy terminal graphics; this is what bpython-curtsies
  was [originally all about](https://github.com/thomasballinger/scottwasright).
* The ability to interface with IPython suggests an external Python process
  being communicated with, which I think is a [great
  idea](https://github.com/bpython/bpython/issues/353).
* Asyncronous autocompletion - I've wanted to make bpython's
  autocompletion asyncronous for a long time! This is terrific.
* Using jedi for autocompletion instead of inspecting objects ourselves -
  interesting! This gives completion within unexecuted code, and additionally
  is sometimes better autocompletion than bpython
* reverse incremental search w/ highlighting - this looks just like what
  I implemented a month ago. It's great!
* Using the normal shell for `raw_input` - smart! Bpython has to be its
  own shell to manage linewrapping of the input, but Python Terminal Toolkit
  appears not to care - and this works just fine! I think it might be an issue
  with fancier resize coping techniques.
* Support for wide terminal characters! bpython "support" doesn't work once a
  line wraps, which is why it can hardly be called support


Things I don't love
-------------------

(all of which are perfectly changeable or matters of opinion)

* Docstrings not displayed, and function parameters are less prominent than bpython
* IPython style prompt (useful for IPython history magicks, but ugly. Same
  with spacing vs the vanilla Python prompt
* Doesn't suspend/background (neither does bpython-curtsies right now, I'm interested
  to see how this gets handled.)
* I don't like the autocompletion colorscheme (wow really nitpicking now - can
  you tell I'm a little sad I didn't write this myself? :)
* Autocomplete doesn't feel as snappy - not sure if this is because it's
  an external process and we're doing IPC (haven't looked at code yet)
  or if it's display code because lines aren't being cached,
  but I've been worried about this happening if we move autocompletion
  to an external process. I think it's worth it, but it feels different.
* autocomplete doesn't re-complete on delete - is this a Jedi thing?
* Literal completion - I wrote my own completion system instead of using
  Jedi, and at rare times it feels nicer, times like `10.0.`
* Decreasing the window width wrecks havok with history.
  This is an issue with bpython-curtsies as well that I want to address, and I'm not yet
  sure that it's worth the hassle. I'm working on terminal emulator testing
  tools so I can try different wrap behaviors, and have written some things to
  predict how different terminals will resize the screen to better compensate
  for this effect.
* No obvious way to go back whole commands vs lines - the Julia repl
  uses ctrl-p/n to go back multiline history entries, arrows to move up and
  down lines - and ctrl-j to run commands, ctrl-m to go down a line.
  (The Julia REPL and an OCaml REPL called utop should both be
  checked out for inspiration if you're writing a REPL.)

Features in bpython not currently in ptpython
---------------------------------------------

Only because the author asked - I don't think all these things are
necessary, and a while ago I mentioned how [nice it might be to break
compatibility](http://ballingt.com/2013/12/21/bpython-curtsies.html).

* Rewind - I personally love this feature. There's some history rewriting
  involved, but I think it's probably worth it.
* Send session to editor - same as rewind.
* Reload modules and rerun session - clears modules out, in the future will
  hopefully use a new Python process. Same infrastructure as rewind.
* Autoreload - same as above plus file watching. Meh.
* I'm using blessings instead of hardcoding vt100 escape sequences - and I
  don't know what the distribution of terminal is, so I don't know how
  much compatibility this buys me. I bet not much.
* No windows support - this is something we got "for free" by using curses
  (with a ton of hard work from Brandon Navra), but in bpython-curtsies
  it's not there. If it were me I'd leave this for later too.
* Autocompletion *above* the cursor - I wouldn't worry about this, and we
  have it off by default in bpython-curtsies - but it can be nice for
  preserving your view of history interspersed with reading long docstrings
* Fish-style right arrow completion. I'm mixed, sometimes I love it, sometimes
  it feels pretty silly. Same with arrow keys doing reverse incremental
  search -ish things with the current line.

A stated goal of Python Prompt Toolkit is to be merged into IPython. I think
this is a worthy goal, but last time I talked to Fernando Perez about this he
was saying they didn't want something that didn't work on Windows in the
general IPython codebase. That doesn't mean that if everyone was crying out
for this it wouldn't be added, but something to be aware of (and perhaps a
reason to use a terminal compatibility layer like blessings if they care about
that too).
