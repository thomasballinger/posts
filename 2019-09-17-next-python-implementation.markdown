---
date: 2019-09-17T23:00:00Z
title: Waiting for the next Python implementation
url: /next-python/
---

The time may be ripe for a new Python implementation.

A [lot](https://www.youtube.com/watch?v=ITksU31c1WY) [of](https://www.youtube.com/watch?v=KDXhu4rxTNY) [keynotes](https://www.youtube.com/watch?v=ftP5BQh1-YM) lately have called for one anyway. They are joined --- informally, not speaking in an official capacity --- by Python core developers in issuing a wakeup call: where is Python in the browser? Where is Python on mobile devices? How could Python be 2x faster?

Barry Warsaw at the [PyCon 2019 Python Steering Council Panel Keynote](https://youtu.be/8dDp-UHBJ_A?t=2288):

> The language is pretty awesome.
> [...]
> The interpreter, in a sense, is 28 years old.

Such a new Python implementation might be faster, work on different platforms, or have a smaller end deliverable.

It might accomplish these goals with a just-in-time or ahead-of-time compiler.

WebAssembly might significantly influence its implementation.

And critically, it might implement a different specification of Python.

---

Wait, what? Will this still be Python?

If the new implementation is useful enough and has a level of compatibility with CPython that the community can deal with, (Brett Cannon said [something like this on a podcast](https://talkpython.fm/episodes/transcript/213/webassembly-and-cpython) a few months ago) then it might somehow be canonized.

This "Optimizable Python" or "Restricted Python" or "Fast Python" or "Static Python" or "Boring Python," subset could, once agreed upon, have its semantics shadowed in CPython in an optional mode.

What might be up for debate? A few suggestions from [Łukasz's talk](https://youtu.be/KDXhu4rxTNY?t=1168):

* eval / exec (compiled in an environment that doesn't allow setting regions as executable, like iOS or (perhaps? I haven't looked) the webassembly spec.
* the complexities and dynamism of the import system.
* metaclasses - I don't know what this enables, but it seems like a concession parts of the community might be willing to make
* descriptors
* dynamic attribute access

---

So now that the Python 2 to 3 transition is wrapping up and the Python language's governance issues have been dealt with, it's time for some dramatic initiatives: let's grab some stakeholders and come up with the parts of the Python spec to mark optional and get this into CPython so we can start porting code again! I propose `python -z` for zoom --- there we go, ZoomPython! --- because I don't see `-z` in python or IPython command line tools. We'll need some syntax like JavaScript's `use strict` to mark code this way, I propose the magic string `# this code zooms`. Can the committee just tell us what the new spec is already?

No! Or as Łukasz Langa says [in response to a a better question after his keynote](https://youtu.be/KDXhu4rxTNY?t=2470),

> "Yes, but the way you get there is to have an alternative platform that informs you what the constrained version of the language should be. If you try to predict the future of what are people are going to need, you're likely to end up with a design that is artificial and not necessarily useful."

So we're back to the hoping and waiting and wondering: where will the implementation proposal for FastPython come from?

---

Despite some [calls for financially support of such an effort](https://youtu.be/ftP5BQh1-YM?t=2898) it seems that leading the prototyping of a new language implementation is not at the top of the priority list for committee. Core developers and language steering committee members seem to believe that this kind of experimental project should come from the outside. (try searching for the word Community in the transcript of [the podcast Brett was on](https://talkpython.fm/episodes/transcript/213/webassembly-and-cpython)) This makes sense to me.

PyPy is the second-most popular Python implementation. It's "bug-compatible" with CPython, including the C extension interface, making it a viable drop-in, faster replacement for many CPython programs. It's an incredible engineering effort, perhaps comparable in scope to the work optimizing JavaScript engines that made that language the fasted dynamically typed language in wide use. If dedicated graduate student and individual hackers, academic funding, and governments grants could make a Python so compliant and fast a Python implementation once, maybe that's where the next implementation will come from too!

MicroPython is closer in design to an imagined implementation of the future: its behavior is [different than CPython in a variety of cases](https://docs.micropython.org/en/latest/genrst/index.html) and includes [the ability to compile individual functions](https://docs.micropython.org/en/latest/reference/speed_python.html#the-native-code-emitter) that do not use features like context managers and generators. MicroPython was initially a Kickstarter-backed effort, then later supported by the Python Software Foundation [as part of its inclusion on the BBC Micro Bit](https://en.wikipedia.org/wiki/MicroPython). The development of MicroPython provides an example of how an alternate implementation might be started by a single individual.

But I think the most likely place for a new implementation to come from is a large company that uses Python and has a specific need for a new interpreter. This belief comes from my time at Dropbox, where I've seen how projects to improve languages can happen at a company of that size: since we had so many programmers working on so much Python code, better Python tooling would be so useful a case could be made for doing it ourselves. At Dropbox this project has been the Mypy Python static type checker, but I could imagine similar projects to write language implementations. (I'm not imagining too hard; Dropbox is also supporting work on mypyc, a Python compiler I'll discuss more in a future post.)

If you are employed at such a company, it's hard for me to know how to help you make the business case, but know that it has been done before! Please consider it.

Where would that be? A lot of companies! Some of my favorite corporate contributions to the Python community have come from Dropbox and Instagram, but Python isn't a niche thing anymore and there must be dozens? hundreds? of companies with idiosyncratic business interests such that a Python implementation that ran in the browser, or ran faster, or ran sandboxed, would save them millions of dollars.

---

In [his inspirational keynote](https://youtu.be/KDXhu4rxTNY?t=962), Łukasz phrased this as a call to action:

> This is where you come in. Truly tremendous impact awaits!

I don't think I'll be one the call-answerers here, but I wish these implementers the best!

I think I support the apparent decision for the search for the next implementation not to be centrally directed; I agree that this can come from the community, and the proof of its usefulness can too. But there is something I think we can do centrally.

Without pre-emptively deprecating Python language features or designating them as optional, Python can be made a more attractive implementation target by making it smaller in a another way: separating the language from standard library.

Glyph proposes moving CPython [toward a Kernel Python](https://glyph.twistedmatrix.com/2019/06/kernel-python.html) for a variety of reasons. I find that case convincing.
