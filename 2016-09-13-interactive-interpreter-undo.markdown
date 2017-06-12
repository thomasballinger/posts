---
date: 2016-09-13T11:00:00Z
suggestedThumbnail: http://ballingt.com/assets/rlundopreview.gif
title: Interactive Interpreter Undo
url: /interactive-interpreter-undo/
---

Undo is my favorite feature of [the fancy python shell I help
maintain](https://github.com/bpython/bpython), and I wish more interactive
interpreters had it. I’ve [previously
written](http://ballingt.com/rich-terminal-applications-2) about the mechanics
of implementing a command line interface that can rewrite its history, but not
about how to teach a system to undo. I’d like to tell you why I like undo, and
I hope afterward you'll consider adding undo to your favorite interactive interpreter.
I’m not going to cover how to write a program or language implementation from
scratch with efficient undo as a design goal, although this is also topic I’m
learning about. (If you have reading suggestions on this topic, please [let me
know!](https://twitter.com/ballingt))
Instead I’ll go over ways to add undo to existing programs and suggest
some hacks you can use to try out undo with many command line interfaces
today.

* [Undo](#undo)
* [Command line interfaces](#command-line-interfaces)
* [Undo Implementation Strategies](#undo-implementation-strategies)
  * [Reconstructing previous states from the current
  state](#reconstructing-previous-states-from-the-current-state)
  * [Reconstructing previous state from the initial
  state](#reconstructing-previous-state-from-the-initial-state)
  * [Recording each state](#recording-each-state)
* [Interactive Interpreters](#interactive-interpreters)
* [Undo in bpython](#undo-in-bpython)
* [Rerunning history in a new process](#rerunning-history-in-a-new-process)
* [Notebook view of a long-lived REPL](#notebook-view-of-a-long-lived-repl)
* [Restoring state with fork(2)](#restoring-state-with-fork2)
* [Hacking fork-on-command into interpreters](#hacking-fork-on-command-into-interpreters)

<br>

## Undo

When I mistakenly delete half of a blog post I’m working on, I’m sure glad I
can hit undo to bring it back. Although leaning on backspace and retyping the
word works well enough for fixing typos, undo allows me to revert arbitrary
changes: if the document suddenly looks funny and I’m not sure why, I can
probably fix it with undo. I’m free to experiment with functionality I don’t
understand, clicking buttons and trying menu items just to see what they do. I
can try out changes without worrying about the overhead of reverting them when
I find I don’t like them.

Having undo means taking an action is only a first attempt at a change, not an
indelible mark. A digital artist friend of mine, Jillian Hazlett, says:

> I typically use undo a LOT when I'm doing digital art. I hardly ever get
> things right on the first try! I typically color a bit, then undo a bit,
> color a bit, undo a bit more. …
> <br><br>
> I also "undo" when I make physical art.
> When I draw in pencil, I tend to erase a lot.
> When I paint, I wipe away mistakes or add another layer on top of them.
> But the undo button is kind of a perfect eraser!
> No smudge marks!
> It allows for easy experimentation with zero risk of "ruining" the piece you're working on.

Undo amounts to time travel in software where document evolution is tied to
the passage of time. Video game console emulators construct the ability to
undo gameplay actions like jumping or shooting out of the ability to save
snapshots of the game state dozens of times per second. Tool-assisted
speedrunning is the process of constructing an optimal playthrough of a game
by playing it until the first mistake, then rewinding to that point and trying
again and again.
With a video game, the end product is not the final state of the game but the
recording of the playthrough; the evolution in time of the document instead of
the document itself.

Once walking forwards and backwards in time becomes second nature,
it can be useful to consider alternate timelines, a branching structure known
as an undo tree, so I can undo a lot, work from there, then change my mind and
go back to the document before the undos. For applications without an undo
tree, saving different versions of a document can serve a similar purpose.
Version control software organizes the process of saving backups over time to
deliver the power of undo and the higher level of tracking documents over long
periods of time.

Undo frees users to experiment, helps them catch mistakes, and allows them to
craft an optimal path from one document state to another.

## Command line interfaces

Command line interfaces are call-and-response, question-and-answer style
programs that take place in the terminal. Their heritage is of electronic
typewriters communicating with faraway computers. The user types something,
and the computer types back. As with many computing technologies, it has been
used to create games:

<script type="text/javascript" src="https://asciinema.org/a/83081.js" id="asciicast-83081" async></script>

Colossal Cave Adventure, by William Crowther and Don Woods. This version of the game was ported to Python by Brandon Rhodes

While playing this game, the player explores a system of caves, gathers
treasure, and dies a lot. Like dying in a Choose Your Own Adventure book, it
would be nice to pretend it didn’t happen and resume the adventure at the last
decision point. What the eleven year old might accomplish with dexterous
fingers lodged into a paperback, Colossal Cave Adventure makes possible
through saved games. At any point on this journey, typing `save
before-the-dragon` saves this game to a file called "before-the-dragon," which
the player can later load to continue playing at this point if the dragon
encounter goes awry.

This is basically undo, with the added inconvenience that the player must
remember to save. The document being restored here consists of the player’s
inventory, the player’s location in the world, and the current status of the
rest of simulated game world. So adding undo doesn’t significantly change the
game beyond making experimentation a bit quicker, and eliminating the need for
constant saving. Let’s imagine we added an undo feature to get around this
inconvenience.

<script type="text/javascript" src="https://asciinema.org/a/83623.js" id="asciicast-83623" async></script>

You can see that the cumulative narrative of the story gets muddled by this
undo command: although the player has no longer eaten the food, the message
that it was delicious is still visible. Similar to restoring a saved game,
undo does not restore text displayed on the screen: the cumulative narration
of the character’s story is only in the player’s head, not on the screen.

This is similar to the limited scope of undo in a word processor: if I
accidentally get stuck in overwrite mode, where typing replaces the content
after the cursor, mashing undo won’t help me to get out of it. Similarly undo
ignores the zoom level, the position of the cursor, and whether print layout
is active. Undo restores the content of the document, not the surrounding
chrome.

It would be nice to increase the scope of undo in this text adventure game to
include the displayed story, restoring not only the character’s position,
inventory, and deadness but also the text on the screen.

<script type="text/javascript" src="https://asciinema.org/a/83625.js" id="asciicast-83625" async></script>

Expanding the scope of undo to include the typed command and its output makes
it more useful: it can now be used to clear typos from the screen and clear
commands that only give the user information instead modifying the
environment, like `inventory` above. Due to some [video terminal emulator
details](http://ballingt.com/rich-terminal-applications), only the text still
on the screen can be overwritten, not text in the scrollback buffer. For now
it’s good enough to modify the text onscreen.

Undo makes exploring more convenient because actions can always be taken back.
While this might not be desirable in terms of game design for a text adventure
game where exploration is supposed to come along with some risk, it
dramatically improves the user interface. Undo makes typos less painful,
status checks less noisy, experimentation cheaper, and crafting an optimal
path less tedious.

## Undo Implementation Strategies

Now that we’ve decided on a preferred interface, we have to worry about how to
implement undo. To restore a previous state, we’ll either need to reconstruct
that state from the current state, rebuild that state from an initial state
plus a history of actions taken since that state, or restore it directly
from a cache of all previous states that have constantly been being saved
to be available for future restores. As in a text adventure game, a
user must wait for undo to finish before continuing to use an interactive
interpreter, so undo should happen quickly enough not to delay the user. As in
a text adventure game, undo ought to accurately recreate the previous state so
as not to mislead the user.

### Reconstructing previous states from the current state

In our first method, actions might be stored along with enough information
about the effect they had in order to later undo that effect. Text editor undo
often works this way. Recording that the user "deleted line 10, which
contained the words 'hello there'" is enough to later reverse this effect. In
Colossal Cave Adventure, this might mean mapping commands to their opposites
and taking note of anything else that happened that turn: if "go west"
resulted in the player going west, falling, dropping all their treasure, and
dying, that action could be recorded and later undone by moving the player
east, reanimating the player, and giving them all their treasure back.

The wide variety of possible commands might make manually implementing the
reverse of each untenable. A more cohesive technique would be to instead
record the state of the document before and after the change and compute the
difference between the two states, then record this difference so it might be
applied later to recreate the old state.

### Reconstructing previous state from the initial state

Alternately, stored actions could be replayed from some initial state to
recreate previous states. Replaying every action but the most recent might
take more time than the previous method, but no additional logic for undoing
every possible command needs to be written. This approach works well when the
same initial state can be recreated and all actions are deterministic. If all
actions aren't deterministic, then the nondeterministic parts need to be
recorded: the reverse debugger [rr](http://rr-project.org/) records
nondeterministic things like the timing and return values of system calls
to replay them later, implementing most of what is needed for a very granular
undo system.
[Redux dev-tools](https://github.com/gaearon/redux-devtools) and the [Elm
debugger](http://debug.elm-lang.org/) similarly record user inputs to recreate
make execution deterministic enough to recreate states.

In Colossal Cave adventure, implementing undo like this would mean saving
a log of all run commands and
replaying all but the most recent. This technique also requires reconstructing or
preserving a known initial state of the program, and being able to quickly
reissue each of a history of commands.

### Recording each state

Finally, snapshots of the state of the program might be saved to be restored
later. Rewind in video game console emulators works by saving a snapshot of
the emulator state every few seconds. Virtual machines that emulate general
use computers can save their state as well, but rarely do so automatically
every few seconds to provide undo.

In Colossal Cave Adventure, this would mean
saving the game after every turn and restoring a previous saved game on undo.
The Python 3 port of Colossal Cave Adventure already implements saving the game
and reloading it, although it does not alter the terminal to replace its
contents with the output of previous commands from the restored game.

Although this method is very flexible, it might take a significant amount of
time to create these snapshots and a significant amount of space to store
them. Both of these costs can be decreased by structural sharing, the
technique of allowing these snapshots to refer each other, sharing some of
their data. Structural sharing is most effective when changes to parts of the
program state are carefully controlled, which is easiest when a program is
built with immutable data structures.

## Interactive Interpreters

An interactive interpreter is a program that waits for the user to type a
command in a programming language, then prints the output of that command.
Interactive interpreters are used to check the syntax or semantics of commands,
to explore libraries, and to accomplish useful work like data munging in an
environment that provides immediate feedback. I use interactive interpreters
to learn and teach because using them keeps my Tested Hypotheses per Minute high:
every sequential step toward a goal is accompanied by a check of my mental
model of the system, and additional side checks are cheap to make.

<div class="code-with-caption">
<pre><code>>>> print(42 + 7)
51
>>> colors = ["purple", "orange"]
>>> print(colors)
["purple", "orange"]
>>> colors.append("red")
>>> print(colors)
["purple", "orange", "red"]</code></pre><br>
A Python interactive interpreter session. Commands like <code>colors =
["purple", "orange"]</code> and <code>colors.append("red")</code> modify the state of the
interpreter. Other commands like <code>print(42 + 7)</code> and
<code>print(colors)</code> do not significantly
modify the state of the interpreter and only inform the user about the current
state.
</div>

I have found undo immensely useful in interactive interpreters for the same
reasons we found it useful in Colossal Cave Adventure. Making exploration more
risk-free and status check commands easier to sweep away makes me more
comfortable exploring. Undo also makes more precise management of
screen real estate possible,
making it easier for an audience to follow along with a live coding
session.
The document which we modify and to which we undo changes to is the
interpreter: what names, like "color", are defined, and to which objects they
refer. However, most interactive interpreters are capable of a wider variety
of actions, some of which affect the world beyond the interpreter state. This
will make implementing undo more difficult than it would have been for
Colossal Cave Adventure.

The wide variety of commands issuable to an interactive interpreter makes
recording each action and the effects of these actions in order to reverse
them later, the first of the three undo strategies, more difficult: it would
be a lot of work to write reverse versions of every command. Finding the
differences between two interpreter states might be possible for interpreters
written from scratch with these inspection capabilities in mind, but typically
this is not easily accomplished.
Modifying an existing interpreter to record these
changes might be possible, similar to the way
[some](http://www.pgbovine.net/SlopPy.html) of [Philip
Guo's](http://www.pgbovine.net)
[modified](https://github.com/pgbovine/Py2crazy)
[Python](https://github.com/pgbovine/PythonCompilerWorkbench)
[interpreters](http://www.pgbovine.net/incpy.html) are instrumented to record
additional information about the state of the interpreter.

The third undo approach is a natural fit for interactive interpreters that
have serialization of session state built in. Matlab and R are languages that,
like Colossal Cave Adventure, have built-in commands to save their current
state to a file which can be used to restore that state later. Implementing
undo for an interpreter of one of these languages might involve saving
snapshots after each run command and restoring the previous one on undo, along
with a bit of extra work to save and restore command outputs.

Some actions taken in an interactive interpreter cause changes of too large a
scope for any of these undo techniques. In this video a programmer is confused
when undo fails for cases involving the filesystem or online shopping.

<iframe width="560" height="315" src="https://www.youtube.com/embed/lUV1zp3YuFU" frameborder="0" allowfullscreen></iframe>

If the scope of undo does not extend beyond the state of the interpreter,
we’ll have to give up on truly undoing commands like deleting a file or
communicating with other computers. A virtual machine or some other
system-level monitor might make changes to the filesystem undoable, but
figuring out a generic way to undo network communication is difficult. Gmail
implements a single undo step for sent emails by recording the user’s intent
to send the email but not acting on it for 30 seconds. Recording intent to
interact with the outside world instead of actually doing so makes sense for
actions that are asynchronous from the users's perspective, like sending an email,
but generally in order to provide the
result of a command as feedback that command needs to take place immediately.
Another approach might be to record intent and predict the result of a
command.
Our undo will simply remove such a command and its associated output from
the screen.

## Undo in bpython

The first place I encountered this style of undo in a command line interface
was with bpython. To try out undo, run `pip install bpython`, run `bpython`,
run a command, and then hit ctrl-r. (reverse-i-search is bound to meta-r
by default, but you can swap these back with a config file)

bpython is an interactive interpreter interface for Python that has had undo
for a long time, and this undo has always erased the undone command and its
output. Initially bpython used an alternate screen display that did not
integrate with other commands in a shell session. I [wrote a new
frontend](http://ballingt.com/bpython-curtsies) to make bpython history
integrate with the parent shell process history, which required some
[interesting terminal
techniques](http://ballingt.com/rich-terminal-applications-2). Through
making these changes and subsequent work on bpython, I became familiar with
and made some changes to its undo functionality.

bpython implements undo using the second of the three previously outlined
strategies: rerunning every command except the most recent in order to
recreate the previous state. When the user requests an undo, a new object
representing an interpreter is created with a clean slate of global variables.
Then the previous state is progressively rebuilt by rerunning all previous
commands except the most recent.

There are some serious issues with this approach. Sometimes side effecting
actions being run again causes unwanted effects. Sometimes rerunning a long
history of commands takes considerable time. Sometimes running a command
doesn’t produce the same output when it is run a second time: read files might
now contain different bytes, a server might return a different response, or a
pseudorandom number generator might produce a different result. Because
bpython runs the new interpreter in the same process, some state shared
between all interpreter objects in a process fails to be reset before
rerunning the command history. This shared state includes the cache of
imported modules (stored in `sys.modules`) and per-process values like the
current working directory and environmental variables.

To mitigate the delay caused by re-executing commands, I added a cumulative
calculation clock to bpython to be used as an estimate for re-execution time.
Once this clock reaches 1 second, the user is given the opportunity to undo
more than a single command at once, to avoid rebuilding states several times
for consecutive undos. Some commands, like calling up interactive help and
waiting for user input, are patched to be less disruptive (not bring up a
pager) and more reproducible (returning the same value they got the first
time) during undo-triggered re-execution. Dealing with commands that
interact with the outside world, also known as IO for Input/Output, is one
of the most interesting topics in implementing undo, but bpython doesn’t do
anything special in the majority of cases.

bpython takes advantage of this otherwise limiting approach to rebuilding
state to implement related features. The module cache not being cleared during
undo allows module reload to be triggered separately by the user with F6,
allowing quickly checking the effect of modifications made to a module.
Because each undo reruns the entire session, it’s possible for the user to
edit that session before it is rerun with F7.

<script type="text/javascript" src="https://asciinema.org/a/84649.js" id="asciicast-84649" async></script>

Before I added it, bpython didn’t implement the first half of the "rebuild
from a known initial state" strategy: it didn’t use a new interpreter object
with new global variables. Interestingly this often worked: rerunning all but
the most recent Python command from the current state often approximates the
previous state well enough be useful.

Because rebinding of variables is allowed in Python, including redefinition of
functions and classes, running previous commands on top of current state
indeed creates a state similar to the previous one. But with this technique
variables would end up defined that shouldn’t have been, which in combination
with Python’s inspection abilities to do things like ask whether a variable
name is defined resulted in inconsistencies in recreated states that could
affect code.

    >>> a = 1
    >>> 'b' in locals()
    False
    >>> b = 2
    >>> <undo>
    >>> 'b' in locals()
    True

The older strategy is still used in cases where the initial state cannot
easily be reproduced, as when it is executed from a debugger with existing
initial state. In this case bpython could either disable undo or provide it in
this more limited way.

This is an important decision. Providing a tool
that doesn’t behave as expected can be worse than not providing it at all.

bpython has a lot of bugs generally; I'm too set in my ways to to notice
many of them, but I'd guess the [Dan Luu
time-to-first-bug](http://danluu.com/everything-is-broken/) would be in the double
digit seconds. I find the payoff high enough and the cost of checking behavior
in regular Python low enough that using it is worth it to me.

Similarly, I appreciate undo so much that I prefer this weaker undo to
not providing the feature at all when the initial state can't be recreated.
However, we’ll see that we can use better undo techniques to avoid
some of these problems.

## Rerunning history in a new process

Although creating a new
[InteractiveInterpreter](https://docs.python.org/3.6/library/code.html#interactive-interpreter-objects)
object improves the reproducibility of the initial state from which undo
states are constructed, global state like the module cache, the current
recursion limit, and other general process state like the current directory
and environmental variables all persist across the creation of a new
interpreter object. The cleaner solution would be to start a new Python
process for each reproduced state for undo. Logging commands and running them
in a new process is a relatively general way to reproduce a process state, and
could be used for many command line interfaces. Here’s a [rough general
wrapper for this](https://github.com/thomasballinger/separate-process-undo)
with very primitive terminal rewriting, and a [Haskell interactive interpreter
wrapper](https://github.com/thomasballinger/hsrepl) that works similarly.

<script type="text/javascript" src="https://asciinema.org/a/84965.js" id="asciicast-84965" async></script>

## Notebook view of a long-lived REPL

You can try out a similar undo manually in a Jupyter notebook. Jupyter
notebooks act as frontends to interactive interpreters in many languages that
run as separate processes called kernels. These kernels can be restarted and
commands, stored in the Notebook interface, can be rerun. The user could
delete the most recent command, restart the kernel and re-execute to implement
a manual undo. Current versions of the notebook even have a
shortcut to restart the kernel and rerun all commands, making this a quick
process.

<iframe width="560" height="315" src="https://www.youtube.com/embed/x2xuoW47pgQ" frameborder="0" allowfullscreen></iframe>

However it's common to make changes to previous commands in Jupyter notebooks
without restarting
the kernel, or to delete commands and their corresponding output if they're
purely informative or not useful.
The notebook interface is strictly more powerful than undo: undo can be
simulated as described above, but the fine-grained control of command output
and ability to edit and rerun individual commands provide many of the same
benefits as undo.
The more limited undo paradigm might be preferred because modifying and
rerunning commands from history out of order can obscure the history of
execution, even removing evidence of the commands that were used to create the
current environment.

Jupyter notebooks trade the display of a log of exactly the commands run so
far for the ability to selectively edit and run commands from history. This
makes it easier to redo a calculation later in a data pipeline without
rerunning earlier steps in the pipeline. In contrast, systems with only undo
always preserve a single path by which the current environment can be
reproduced.

I think Jupyter notebooks are amazing. I’ve used them with Haskell, R, Ruby,
and JavaScript in addition to Python. If you're not so far convinced by my
exhortations to use undo, consider ignoring the rest of this post and proceeding
directly to using and contributing to the Jupyter kernel for your favorite language.
I would only consider arguing that undo is preferable in terminal interfaces to
the Jupyter kernels, or for situations in which it's difficult to keep a notebook
reproducible.

## Restoring state with fork(2)

Using a separate process for each undo allows a cleaner initial state, but
rebuilding state from there can still cause unwanted side effects, and we’re
still left with nondeterminism in interactions with the world beyond our
process: changes to the contents of files, servers that might return different
responses, etc. If only we had snapshots of the process, it wouldn’t be
necessary to risk causing effects outside of the process again or risk
requests getting different responses each time undo is used. It would be
awfully nice to make copies of entire processes, and restore those copies as
needed.

Fortunately making a snapshot of a process is possible on unix systems with the fork system call!

<script type="text/javascript" src="https://asciinema.org/a/85182.js" id="asciicast-85182" async></script>

Each time [this Python
interpreter](https://github.com/thomasballinger/forkundo) asks for input, it
makes a copy of itself with fork. One of these copies goes to sleep for a bit
while the other executes the entered command. If the user types "undo," then
that process dies and the other wakes up and resumes execution at that
earlier point.

    import code, os, sys
    def read_line(prompt=""):
        while True:
            s = input(prompt)
            if s == 'undo':
                sys.exit()  # restores prev. state in parent
            if os.fork() == 0:  # this is the child
                return s
            else:  # this is the parent
                os.wait()  # wait for child process to die
    console = code.InteractiveConsole()
    while True:
        src = read_line('>>> ')
        console.runsource(src)

Fork works well enough for simple programs, but [fork is
dangerous](http://www.evanjones.ca/fork-is-dangerous.html). Once the process
being forked has multiple threads all bets are off.
And a good new bet is that things won’t work.

One of the difficulties of saving snapshots of an interpreter is the storage
space required to keep track of them. Since these copies of processes are
stored in RAM instead of being serialized to disk, there’s even less space
available to store them. The copies of the process are quite similar to each
other, and fork takes advantage of this similarity: portions of a process are
only copied when one copy or the other makes a change to a portion of memory,
called a page. This "copy-on-write" strategy means waiting to copy over of a
block of memory to another block until one of the processes writes to that block.
This means
the amount of additional space required per copy depends on how many pages of
memory are modified to execute a single command. Unfortunately, simple
commands may result in changes to a large number of pages in interpreters not
optimized for the fork system call. See [this
article](http://patshaughnessy.net/2012/3/23/why-you-should-be-excited-about-garbage-collection-in-ruby-2-0)
on how this behavior was improved in version 2.0 of Matz's Ruby Interpreter.
It suggests this technique will work much better with interpreters that go
out of their way to make forking efficient.

On my machine each fork of Python 2.7 takes between 200 to 1000 KB for
"typical" interactive use, that is, not creating or loading large data
structures. A better system might be more careful about when to fork, perhaps
only maintaining enough copies of the process to support a few undos.

## Hacking fork-on-command into interpreters

What if we wanted to use forking but not mess with interpreter internals?
Command line interfaces often follow a pattern of calling a function from the
GNU Readline library to get their next line of input. It’s possible to
intercept these calls to readline and substitute a patched version that uses
fork to copy the process, in a manner similar to the Python code above. This
hijacking is accomplished by encouraging the executable to load a different
shared library.
On Linux this is done with
[LD_PRELOAD](http://jvns.ca/blog/2014/11/27/ld-preload-is-super-fun-and-easy/),
on OS X with `DYLD_LIBRARY_PATH`. This hack requires that the interpreter executable loads the readline library dynamically, or else recompilation of the executable is required.

Since readline knows when undo is happening, it knows when the terminal
display should be saved or restored by erasing some rows of output.
The interpreter is run in a pty to cajole it into allowing its terminal
output to be recorded
and our patched readline library can send a message to the process controlling
the pty in which the interpreter is running announcing when to record
or restore terminal states.
Now we have the undo we set out to obtain for many interactive interpreters
without modifying them each individually.

<script type="text/javascript" src="https://asciinema.org/a/85535.js" id="asciicast-85535" async></script>

This project is called [rlundo](https://github.com/thomasballinger/rlundo)
after the useful wrapper program rlwrap.
It's a fun hack (and if you want to play with it, [it could use some
love](https://github.com/thomasballinger/rlundo/issues)).
But it's probably never going to be a robust solution.
rlundo is a way to try out undo in your favorite interpreter
so if you like it you can look at adding it in a less hacky way :)

<a href="https://twitter.com/ballingt" class="twitter-follow-button" data-show-count="false">Follow @ballingt</a><script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

---

Thanks to Ezekiel Smithburg, Leah Hanson, Julia Evans, and Kamal Murhubi
for feedback on this post. Thanks to Erin Murray, Vaibhav Sagar, Sarah Ransohoff,
and Katherine Erickson for edits. Thanks to Mary Rose Cook for feedback on
an early version of this post.
