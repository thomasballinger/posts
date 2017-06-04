---
layout: post
title:  "Native scrolling in bpython: bpython-curtsies"
date:   2013-12-21 11:12:17
alias:  /2013/12/21/bpython-curtsies.html
tags: bpython
---

I've mostly finished up work on an alternative frontend for bpython.

Updated: this is now the default bpython frontend,
installed with `pip install bpython`
and then run with `bpython`. To use the old frontend, use `bpython-curses`.
The rest of this post refers to this new frontend as `bpython-curtsies`.

`bpython-curtsies` is bpython with native terminal scrolling:

![bpython-curtsies scroll demo]({{ site.url }}/assets/bpython-curtsies-scroll-demo-large.gif)

It also has send-to-editor (F7 by default) and an improved version of
good old bpython rewind: `raw_input` prompts are saved so they don't need
to be entered again, and a new environment is used instead of the old one,
so variable bindings are actually undone.

![bpython-curtsies undo, editor, and caching stdin demo]({{ site.url }}/assets/bpython-curtsies-undo-editor-prompt-save-demo.gif)

Since it's not limited to keys curses can detect, there are more keybinds:

![bpython-curtsies keys demo]({{ site.url }}/assets/bpython-curtsies-demo-large.gif)

[Curtsies](https://github.com/thomasballinger/curtsies) is a terminal wrapper I'll write more about in another
post.

Build your own lightsaber
-------------------------

Having rewriten so much of the code, there's very low overhead for me to add new
features. A few weeks ago I added the ability to edit the current REPL session in an
external editor, and used the feature a ton that day while answering questions
some Hacker Schoolers had reading the excellent
[Python Essential Reference](http://www.amazon.com/Python-Essential-Reference-4th-Edition/dp/0672329786)[^1].

Sam Aaron, creator of Overtone and Emacs Live, advocated building one's own tools in a [talk I listened to](http://www.infoq.com/presentations/Live-Programming) the other day.
I'm referencing him mostly because I wanted to use this section title.
As he suggests, I've found using a tool I created daily rewarding and powerful.
I've added other features on a whim that aren't in the current pull request:

* vim-style abbreviations: `improt` -> `import`
* automatic import (try to import libraries when name errors occur)
* upload to [Online Python Tutor](http://pythontutor.com/) (available as a [standalone pastebin helper](https://gist.github.com/thomasballinger/8106585))
* Demo mode: display of what key was just pressed, and scripts that
  play back bpython sessions key by key
* Features specific to the Python turtle graphics library, including better
  undo.

These features may not belong in bpython, but at various times they've been
useful to me.

Contribute!
-----------

Update: It's merged! Just grab the lastest bpython from
[github.com/bpython/bpython](https://github.com/bpython/bpython)
to start hacking! There's plenty of work to be done though, see the main
[bpython issue
tracker](https://github.com/bpython/bpython/issues).

If I were to write another REPL, I might just write it from scratch, but it's exciting that
my work might be used by the existing bpython user base, and the framework
provided by the existing code and feature set was
helpful in structuring and scoping the project.

[^1]: By the way, I've written [a little quiz]({% post_url 2014-05-02-pyquiz-intro-quiz %})
    that acts as an advertisement for Python Essential Reference.
