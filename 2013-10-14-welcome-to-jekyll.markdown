---
layout: post
title:  "Edit bpython session in an external editor"
date:   2013-10-14 11:12:17
categories: bpython python
---

I'm working on an alternative frontend for bpython - it's installable with
`pip install hg+http://hg.bitbucket.org/thomasballinger/bpython@scroll-frontend`,
and called `bpython-scroll`.

Having rewriten so much of the code, there's very low overhead for me to add new
features, and today I added an option to edit the current REPL session in an
external editor. I've already used the feature a ton today while answering questions
Hacker Schoolers had reading the excellent
[Python Essential Reference](http://www.amazon.com/Python-Essential-Reference-4th-Edition/dp/0672329786).

Editing session history works like the bpython rewind feature - after editing, it's just
running the commands in your history again (though in my version this happens
in a new interpreter session), so environment affecting things like file io
and network connections don't do well with this strategy.

I also fixed history persistance, and to handle this new
"edit session in editor" mode it's being added again.
