---
layout: post
title:  "Richer command line interfaces"
date:   2016-07-21 9:30:00
categories: bpython
---

This post is preceded by a description of [two types of command line
interfaces](/rich-terminal-applications). The more command line-like of these
two types integrates into a normal shell session (`python`, `telnet`, `bash`)
instead of using the alternate screen to present a fullscreen text user interface
(`top`, `vim`, `emacs`).
Sometimes programs blur this boundary with fancy formatting in a line-oriented
command line interface, carefully avoiding overwriting
previous user inputs and command outputs.
On-keystroke autocompletion, multiline editing, inline documentation and
syntax highlighting can be implemented like this, making command line
interfaces more useful.

<script type="text/javascript" src="https://asciinema.org/a/79153.js" id="asciicast-79153" async></script>

But this can be taken further than it usually is!
Here are three features that I'd like to see
used in more programs (or I'd like to better understand their tradeoffs).

Checking vertical space before scrolling
========================================

When writing this kind of
program it's useful to consider two separate canvases available for drawing
the current line and interface widgets like autocomplete suggestion:
space that can be drawn in without causing the terminal to scroll down, and
space that can be drawn in which would cause the terminal to scroll, but would
keep the cursor onscreen.

<img src="/assets/simpleterminaldiagram.jpg" alt="" style="width: 100%"/>


Applications
------------

Most autocompletion doesn't seem to take advantage of this, providing the same
number of completion suggestions regardless of the amount of space left in the
terminal.

Fish seems to force 5 rows by default, which may force push terminal content
up or may not be enough to fill the window:

<script type="text/javascript" src="https://asciinema.org/a/80488.js" id="asciicast-80488" async></script>

while Python Prompt Toolkit does a nice job, still forcing 5 rows:

<script type="text/javascript" src="https://asciinema.org/a/80489.js" id="asciicast-80489" async></script>

bpython measured the visible space but used too large a minimum completion box
size until a few weeks ago:

<script type="text/javascript" src="https://asciinema.org/a/79748.js" id="asciicast-79748" async></script>


Implementation
--------------

The larger of the two areas available for completion suggestions areas in the
diagram above is as tall as the height of the
terminal, which in a unix-y environment is a single cryptic `TIOCGWINSZ`
`ioctl()` call away.
But figuring out how much space is available without scrolling depends on
where in the terminal the prompt has been printed. Finding this is a bit
tricker.
We can get at it by querying the terminal for the cursor's
position with `\x1b[6n`, but it's
a bit of a hassle because the response comes
back from the terminal on stdin and needs to be distinguished from user input.

    $ echo -ne "\x1b[6n"; read -sd R POS
    $ printf "%q\n" $POS
    $'\E[14;1'  # cursor is in row 14, column 1

Once we know where the cursor is we can do some row/column arithmetic to find
the region that can be used for completion without causing a scroll.
Keeping previous lines of history visible is generally useful, and
particularly so when livecoding for a group that is trying to follow along.

Handling terminal resizes
=========================

Many terminal emulators allow you to resize a session already in progress --
something that would have made no sense in a traditional video terminal!
Each time the user changes the terminal size a SIGWINCH signal is sent to
the program and the terminal emulator moves the terminal contents around.
The signals may be debounced a bit as they pass through
additional layers of terminal emulation like tmux and GNU Screen.

Fullscreen, alternate screen-using programs don't care how this content
gets moved around because they can completely rerender after the resize.
But a line-oriented command line interface should care more:
if the resizing behavior can be
predicted editing artifacts can be prevented from polluting terminal history.

Applications
------------

Here's an example of a pathological case: a wrapped cursor causes the multiline
input to be displayed a row too low on each resize:

![](/assets/ipythonresize.gif)

The desired behavior here would be not to fill the screen with repeated
text, but this is surprisingly difficult to achieve.
Tools I've worked on suffer from similar problems. I've worked hard to
minimize them, but I haven't been able to get it anywhere near perfect,
particularly for cases like
"the user quickly narrows, widens, then narrows the terminal."
Single resizes of any kind aren't so bad, and the result of any number of
consecutive height changes is predictable.

Resize behaviors have dramatic (history rewrapping) and
subtle (which lines are shown or hidden when changing height) differences in
different terminal emulators, so querying the cursor position after
resize is useful for checking to see how content has been moved around.

Implementation
--------------

I spent a considerable amount of time trying to get this right in bpython, to
little avail. Some later attempts were less complete but more promising.
My first suggestion is to debounce window size changes: wait to redraw until no new
SIGWINCH signals come in for a bit. This sacrifices responsiveness in order to mess
up history less: if you're wrong, at least you're only wrong once, ruining a
few rows instead a few dozen.

Changing the height of a terminal window moves text is pretty predictable, and
the few edge cases can be detected by checking the cursor position after
the move.

An example of different behavior requiring a cursor check to detect:

<img src="assets/vertical-resize-iterm.gif" alt="" style="width: 100%"/>
<img src="assets/vertical-resize-terminalapp.gif" alt="" style="width: 100%"/>

Changing window width on terminal emulators that wrap text is tricker,
particularly while text from a previous program remains visible, since
the length of these lines can't be known. But again, checking for the new
cursor position should generally give enough information.

To get an implementation of this right, it would be helpful to test multiple
popular terminals for wrapping behavior.
I briefly looked into orchestrating iTerm for this but found tmux
easier to script and more portable. For that project I use tmux check that ASCII diagrams
correctly describe resizing behavior:

    def test_wrapped_undo_after_narrow(self):
        self.assert_undo("""
                                 +------+  +------+
        +------+  +-----------+  |$rw   |  |$rw   |
        +------+  +-----------+  +------+  +------+
        |$rw   |  |$rw        |  |>1    |  |>1    |
        |>1    |  |>1         |  |>stuff|  |>@    |
        |>stuff|  |>stuff     |  |abcdef+  ~      ~
        |abcdef+  |abcdefghijk+  |ghijkl+  ~      ~
        |ghijkl+  |lmnopq     |  |mnopq |  ~      ~
        |mnopq |  |>@         |  |>@    |  ~      ~
        |>@    ~  ~           ~  ~      ~  ~      ~
        +------+  +-----------+  +------+  +------+
        """)

If there were a really solid implementation of this it would be great, but it
alone might not be worth the effort. Users of command line interfaces are
trained to hit ctrl-l to request a clear or redraw as a workaround, and are
already used to aggressive resizing messing up their history.

Rewriting previous lines
========================

If we remember how many lines have been printed, we can know it's ok to write
to those lines. For example, this way autocompletion suggestions could be shown *above* the
current line instead of below:

    $ echo hello
    hello
    $ prompt_with_autocompletion_above
    > s = "a"
    > s += "b"
    > s += "c"
    > s
    'abc'
    > s.en|

---

    $ echo hello
    hello
    $ prompt_with_autocompletion_above
    > s = "a"
    > s += "b"
    > s+--------+
    > s|encode  |
    'ab|endswith|
    > s.en      |
       +========+

To do this, it's necessary for the program to model how much of the terminal can be
written to and restored:

<img src="/assets/rewritehistorydiagram.jpg" alt="" style="width: 100%"/>

Careful attention to the terminal window size is even more important now
that there are so many potentially wrapping lines.

Applications
------------

In order to keep more history onscreen, it can be nice to show a completion
box above instead of below text:

<script type="text/javascript" src="https://asciinema.org/a/79745.js" id="asciicast-79745" async></script>

This is off by default in bpython because it's surprising and a bit annoying
if you wanted to see previous lines. But the more general feature of writing
to
previous lines enables all sorts of features like editing previous lines of a
REPL session, reloading changes to a module, and undoing the last command:

<script type="text/javascript" src="https://asciinema.org/a/79746.js" id="asciicast-79746" async></script>

Undo in this kind of stateful, side-effecting tool is pretty far out, a
wackiness I hope to write about soon. But rewriting the text
above the current line can also be useful for side effecting programs that
can't easily recreate previous states: it's possible to clean up error
output, collapse long command outputs, and annotate previous commands currently
onscreen based on the current command. Alternate screen-style interfaces
can be displayed in portions of the screen above the current line this way
in addition to below.

Implementation
--------------

As required for dealing with window size changes, our command line program
must be responsible responsible for simulating normal terminal
behavior like wrapping in order to predict the contents of the terminal so
they can be rerendered.

Each time I've implemented this I've written the terminal emulator
lines wrapping calculations myself.
After a too-naive approach [the first time](http://ballingt.com/bpython-curtsies)
led to [bugs](https://github.com/bpython/bpython/issues/574), I'm most
interested in an approach using a headless terminal emulator to make these
decisions. I haven't yet found one that has implements linewrapping quite the
way graphical terminal emulators do and would appreciate suggestions.

But even ignoring terminal window resizes completely sometimes produces
useful results, in this [simple Haskell REPL
wrapper](https://github.com/thomasballinger/hsrepl):

<img src="/assets/hsrepldemo.gif" alt="" style="width: 100%"/>

Of course there's a limit to how much of history can be changed: the
scrollback buffer cannot be modified! So bpython writes a warning offscreen
that scrollback history is no longer consistent with the history that appears
after it:

<img src="/assets/bpython-inconsistent-history.gif" alt='' style="width: 100%"/>

Call to action
--------------

If you know of other programs that use these techniques, I'd love to hear
about them. If you know why they aren't common, I'd love to hear that too!
And if you try implementing them, I'd love to chat about implementation.
It'd be great to see them in a tool like readline we can all just
use, the way Python Prompt Toolkit makes [adding so many fancy features to a REPL very
simple](https://github.com/jonathanslenders/python-prompt-toolkit/tree/master/examples/tutorial).

---

Thanks to Amjith Ramanujam, author of the wonderful
[pgcli](https://github.com/dbcli/pgcli) and other interactive REPLs, for
feedback about these features and for pointing out that Python Prompt Toolkit
already implements the first one.
