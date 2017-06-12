---
categories: bpython
date: 2016-07-09T11:12:17Z
title: Two types of command line interfaces
url: /rich-terminal-applications/
---

Command Line Creature Comforts
------------------------------

When a command line utility writes to stdout, sometimes that output is going
to be piped to another utility or redirected to a file. And sometimes the
input it reads from stdin will be coming from a file or another utility.
But other times that output is just going to be displayed by a terminal emulator,
and that input is going to be provided by a human typing.

Command line utilities can make allowances for the human input and output
use cases: here's `ls` using multiple columns (and on your machine likely
color, too) when its output isn't being piped anywhere.

    $ ls > tmp.txt; cat tmp.txt
    diagram.png
    image.jpg
    notes.txt
    $ ls
    diagram.png     image.jpg       notes.txt

This article skims over precisely how terminals can be made to act differently
in this case; if you're not familiar and uncomfortable with handwaving, you
might duck out and have fun with some of these resources first:

* [Terminal Whispering](https://www.youtube.com/watch?v=rSnMoClPH2E) (video)
* [Build your own Command Line with ANSI escape codes](http://www.lihaoyi.com/post/BuildyourownCommandLinewithANSIescapecodes.html)
* [ANSI escape codes](https://en.wikipedia.org/wiki/ANSI_escape_code)


Some interpreter commands like `bash`, `python`, and `node` switch completely to interactive mode to
provide a REPL when run without a target, and even use `readline` to provide autocompletion,
completing the rest of a word if only one possible completion exists and listing all
possible completions if more than one does.

<script type="text/javascript" src="https://asciinema.org/a/79156.js" id="asciicast-79156" async></script>

Two approaches to history
-------------------------

Unfortunately, with limited terminal emulator real estate these suggestions
can quickly move previous commands off the screen.
This problem is somewhat mitigated by the
scrollback buffer history of our teletype-inspired terminal emulators:
although we no
longer automatically end up with a paper history of our consultations
with the console,
an analogous virtual readout is only a mousewheel flick away.
But it's not ideal be lowering the signal to noise ratio of this scrollback
buffer history by polluting it with completions.

Since the advent of video terminal, we've been free of the typewriter-tyranny of
only moving right (normal characters), down (`\n`), and to the left
(`\r`, `\b`). Now, ANSI escape sequences allow us to write to any portion of
the traditionally 80-column-by-24-row video terminal. But this freedom
does not extend to the scrollback buffer; once a row is pushed off the
screen, it becomes indelible.

This history feature only makes sense in the line-oriented paradigm, so
most so-called fullscreen programs which use the terminal emulator as a canvas
rather than a souped-up typewriter use the "alternate screen" where history
is not preserved and writing a newline on the bottom row does not cause a
frame shift.

<script type="text/javascript" src="https://asciinema.org/a/79147.js" id="asciicast-79147" async></script>

As such, exiting `vim` or `emacs` does not add a full terminal height of
text editor detritus from a session with one of those text editors. Their interaction
takes place on the alternate screen. In `vim`, output from `!`-preceded shell
commands (which do fit the line-oriented paradigm) will be added to the regular,
history-preserving terminal session.

The two features of scrollback buffer and rich terminal interaction are
therefore at odds, and this conflict comes to a point in
command line interfaces: interactive call-and-response sessions like
interactive shells, interactive interpreters, or any other program
with its own prompt whose execution encompasses several inputs and
corresponding outputs. That barrage of information provided by text editors
is useful, but, like autocompletion suggestions, would further pollute our
history.
The fullscreen app approach eschews the scrollback buffer entirely,
not contributing to it but not mucking it up, either. By contrast, interactive
line-oriented sessions typically do store their history.

<script type="text/javascript" src="https://asciinema.org/a/79149.js" id="asciicast-79149" async></script>

A hybrid approach is to use the line-oriented method on the alternate screen,
simulating a regular history-preserving session by scrolling content offscreen to
make room for additional lines -- and then dumping all of this history
when the program finishes to preserve it.

<script type="text/javascript" src="https://asciinema.org/a/79150.js" id="asciicast-79150" async></script>

Since the session has been added to the regular terminal it will end up in the
scrollback buffer and be viewable later,
but previous history cannot be viewed by scrolling up while that program is
running.

Not mucking up history
----------------------

The basic rule of etiquette for history-recording command line programs -- that
is, programs which do not use the alternate screen -- is to not alter
previously run commands or their output.

<script type="text/javascript" src="https://asciinema.org/a/79151.js" id="asciicast-79151" async></script>

Writing ordinary text makes following this pretty easy; again, the tyranny of
the typewriter allows only writing characters, moving to the next line, and
possibly editing the current one with carriage return and ASCII character 8,
backspace. It's hard to mess up history like this.

The often-used readline library adds hotkeys for jumping around the
current line by reading input into userspace character-by-character
(instead of waiting for a newline) and repainting that line of text to the
terminal immediately.

This task is more difficult than it appears because
a single logical line might be split between multiple rows. Since
capabilities for redrawing the terminal are based on display rows and columns
instead of logical lines, the readline library has a tricky job to do when
an edit -- say, the deletion of word with ctrl-w -- affects other rows before and
after. It must use what it knows about the current width of the terminal
to hop the cursor around to rebuild the current line.
In fact, it's pretty easy to confuse readline: start a program that uses
it (like Python), drag the terminal window smaller, then type a long line of
text.
I've found libedit, usually used in place of readline of OSX, more difficult
to confuse; regardless, it's a difficult task.

I'll push out the difficulties introduced by resizing terminal windows and
reflowing onscreen history to another article. Let's ignore them for now.

Tools like fish and utop take a sensible approach: although it's rude to alter
history above the current line, rows *under* the current line can be used as
temporary space for any transitory information the user might want to be hit
with.

<script type="text/javascript" src="https://asciinema.org/a/79153.js" id="asciicast-79153" async></script>

Multiline editing can even be implemented in this way. Here's IPython
5, which now uses the excellent [Python Prompt Toolkit library](https://github.com/jonathanslenders/python-prompt-toolkit):

<script type="text/javascript" src="https://asciinema.org/a/79154.js" id="asciicast-79154" async></script>

It's very important that these autocompletion menus appear *below* the
current line of text. Terminal emulators provide no way to *read
the contents of the screen*. Because we can't read that content to save it, if
we wrote over it, the best cleanup we could do afterwards would be to
erase the completion with spaces, leaving a gap.

Although we have no way to know what
content appears above the top row of the current line in the first prompt of a
command line interface, by the second prompt at least some of that content
is knowable: it was put there by this very same program, one while loop
iteration ago!

In [part 2](/rich-terminal-applications-2) we'll look at using knowledge of
what had just been written modify previous rows
and see how terminal window size changes, history rewrapping, and differences
in terminal emulator behavior make this difficult. Here's a teaser:

<script type="text/javascript" src="https://asciinema.org/a/79155.js" id="asciicast-79155" async></script>
