---
layout: post
title:  "PyGotham bpython lightning talk"
date:   2014-08-16 18:35:00
---

The Python REPL is great! You can try out code before putting it in your
program and get information about types and objects:

<script type="text/javascript" src="https://asciinema.org/a/11518.js" id="asciicast-11518" async data-size="small" data-speed="1.7"></script>

A while ago some people created an even better REPL - you probably haven't
heard of it:[^1]

<script type="text/javascript" src="https://asciinema.org/a/11519.js" id="asciicast-11519" async data-size="small" data-speed="1.5"></script>

Then a fellow named Bob created bpython. It does some cool things. I wrote
bpython-curtsies, an alternative frontend to bpython, that's going to be what
you get when you type bpython in the next release (0.14, coming in the next
few weeks hopefully). You can install stable bpython now with

    pip install 'bpython[curtsies]'

but to use bpython curtsies you need to type "bpython-curtsies" instead of
"bpython" at the prompt until we release bpython again in a few weeks.

<script type="text/javascript" src="https://asciinema.org/a/11522.js" id="asciicast-11522" async data-size="small" data-speed="1.2"></script>

(being able find the source code of a c function in the first place is due to
the efforts of Puneeth Chaganti, not bpython:
[cinspect](https://github.com/punchagan/cinspect)
lets you view c python
source, and comes with a convenient ipython monkey-patch)

bpython makes editing your current session easy: ctrl-r undoes the last line,
and F7 sends the whole session to an editor.

<script type="text/javascript" src="https://asciinema.org/a/11523.js" id="asciicast-11523" async data-size="small" data-speed="2"></script>

You can reload (a pretty safe reload, actually importing all the modules
you imported again, using a new interpreter object) with F6,
and turn on automatic reloading with F5:[^2]

<script type="text/javascript" src="https://asciinema.org/a/11524.js" id="asciicast-11524" async data-size="small" data-speed="1.6"></script>

So try out bpython! (some features here are only in master - either download
the [latest bpython](https://github.com/bpython/bpython) or wait a few weeks
to be able to use it)

[^1]: I think IPython is terrific, and this was a laugh line - 
    most Python developers have heard of it because IPython because it's
    a super awesome, successful, popular library.

[^2]: I made an offhand remark here about IPython-style reloading not being safe - 
    this wasn't really fair, and I didn't back it up at all. 
    What I meant was that `%run`, one way to reload modules in IPython,
    loads the modules globals into the local
    namespace, which is a little scary: if you've deleted globals in
    the module, you'll have the old ones still around. There are also problems
    with a normal `reload(module)` command: objects
    you've already imported with `from module import thing` syntax
    won't be reloaded, and recursive reloads are an issue: reloading
    a module doesn't reload the modules it imported (IPython has
    [something](http://ipython.org/ipython-doc/dev/interactive/reference.html#dreload)
    for this). Edit: It turns out
    [autoreloading](http://ipython.org/ipython-doc/dev/config/extensions/autoreload.html)
    in IPython is pretty awesome, and fixes many cases of the
    `from module import func` or `from module import class` problems - thought
    constants like `from module import MAX_LENGTH` is still a problem.
