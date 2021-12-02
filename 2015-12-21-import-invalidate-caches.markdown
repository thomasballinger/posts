---
categories: bpython python
date: 2015-12-21T14:00:00Z
title: Remember to use invalidate_caches() in Python 3
url: /import-invalidate-caches/
---

{{< tweet user="ballingt" id="679016654651703296" >}}

I was getting intermittent failures of a
test that checked that a feature of
[bpython](http://bpython-interpreter.org/) was working.[^feature]
The test was failing on my computer
despite [passing on Travis CI](
https://travis-ci.org/bpython/bpython/jobs/94126769#L536),
a service that automatically runs the tests of
the project each time I send changes to GitHub.
The test only failed if another test had been run
right before it, so running this other test was somehow
changing the environment in which the failing test ran.

I whittled down the combination of the two tests to the
following:[^source]

{{< highlight python >}}
import os, sys, shutil

if os.path.exists('tempdir'):
    shutil.rmtree('tempdir')
os.mkdir('tempdir')
sys.path.append('tempdir')

temp = open('tempdir/tempmod1.py', 'w')
import tempmod1

temp = open('tempdir/tempmod2.py', 'w')
import tempmod2
{{< / highlight >}}

In Python 3.3 or later an import error is raised by this code:

{{< highlight python >}}
Traceback (most recent call last):
  File "tmp1.py", line 12, in <module>
    import tempmod2
ImportError: No module named 'tempmod2'
{{< / highlight >}}

I found a few more clues:

1. `os.path.exists('tempdir/tempmod2.py')` evaluates to True on any line after
   that file is opened.

2. I could comment out the `import tempmod1` import statement to make the next one succeed. (!)

3. I could add a `time.sleep(.5)` between `import tempmod1` and `import
   tempmod2` to make the second import succeed about half
   the time, 10% of the time for .1, and always for times greater than 1 second.

Take a moment to think if you like.

---

I didn't discover the third point above until after understanding the problem but it's
too good of a hint not to present here. This understanding came after running the
script with `python -vv`[^vvv] which reported that an instance of the
`importlib.machinery.FileFinder` class was responsible for
determining whether this source file existed, the
[documentation for which](https://docs.python.org/3/library/importlib.html#importlib.machinery.FileFinder)
described exactly the behavior I was seeing and offered this solution:

<blockquote>
If you are dynamically importing a module that was created since the
interpreter began execution (e.g., created a Python source file),
you may need to call
<a href="https://docs.python.org/3/library/importlib.html#importlib.invalidate_caches"><code>invalidate_caches()</code></a>
in order for the new module to be noticed by the import system.
</blockquote>

# Why does the second import failing depend on the first?

The first time an entry of `sys.path` is searched during the import of a Python module a
`FileFinder` object is created for that path and stored in the
`sys.path_importer_cache` mapping of path entries to finder objects.

Each `FileFinder` caches the list of files their directory contains for faster
lookups in the future -- after all, every time a new module is imported
that directory might be searched again to see if it has a file with a name
matching the module being searched for.

Because a new file might have been created in that directory since the
last time its contents were checked, each time the contents of
the directory are stored the directory's
[stat mtime](https://en.wikipedia.org/wiki/Stat_(system_call))
is recorded:
the time at which the contents of that directory were last modified.
When the next import statement causes another search through these finders
the old list of files for each can be used if the mtime for that directory
has not changed.
If it has, the contents of the directory and the new state mtime
are recorded again.

In our case, the contents of our `tempdir` directory have been cached
by the first import statement, and files created subsequently are not
present in this cache.

# Why does time.sleep(1) fix everything?

Because cache invalidation is done by stat mtime, which has
has resolution of one second.

[pitrou found similar failures](http://bugs.python.org/msg153607) in
the Python tests:

>There are a couple of test failures (in test_import, test_runpy,
test_reprlib). Apparently this is due to false positives when comparing
directory modification times. If I sleep() enough between tests, the
failures disappear...
I wonder if this could be a problem in real-life (say, someone compiling
some DSL to Python code on-the-fly and expecting import to pick up
without any timestamp problems).


There's a description of exactly this behavior in the last paragraph of
the [`FileFinder`
docs](https://docs.python.org/3.3/library/importlib.html#importlib.machinery.FileFinder):

>
The finder will cache the directory contents as necessary, making stat calls
for each module search to verify the cache is not outdated. Because cache
  staleness relies upon the granularity of the operating system’s state
  information of the file system, there is a potential race condition of
  searching for a module, creating a new file, and then searching for the
  module the new file represents. If the operations happen fast enough to fit
  within the granularity of stat calls, then the module search will fail. To
  prevent this from happening, when you create a module dynamically, make sure
  to call <a
  href="https://docs.python.org/3/library/importlib.html#importlib.invalidate_caches"><code>invalidate_caches()</code></a>.

# Why Python 3.3?

Because that's the first release that had [pitrou's patch that added
this caching](http://bugs.python.org/issue14043).
The import system [had recently gone through an overhaul](
https://docs.python.org/3/whatsnew/3.3.html#using-importlib-as-the-implementation-of-import)
that made optimizations like this easier to implement because the
code was now in Python.

# Why was the test passing (at least sometimes) on Travis CI?

I don't know! It seems to be the original test suite's use of temporary files
that introduces this discrepancy between systems. While the code above
reliably fails on an OS X machine and the Linux machines I tested it on,
code below always fails on OS X and *usually* succeeds for me on a Linux machine.

{{< highlight python >}}
with tempfile.NamedTemporaryFile(suffix='.py') as temp:
    path, modname = os.path.split(temp.name)
    sys.path.append(path)
    __import__(modname.replace('.py', ''))

with tempfile.NamedTemporaryFile(suffix='.py') as temp:
    path, modname = os.path.split(temp.name)
    sys.path.append(path)

    __import__(modname.replace('.py', ''))
{{< / highlight >}}

If you know what's going on, please let me know!

---

When I get into the second hour of a debugging journey
in Python a small, persistent hope appears
that I've found a bug in the interpreter.
I know the odds: in eight years of using CPython I've found
[only](https://bugs.python.org/issue16782)
[two](https://bugs.python.org/issue21217)
that hadn't previously been reported.
But for me it's a helpful incentive to figure out the
root cause of a problem instead of going with a workaround.

I informally tend to think of anything working in Python 3.2 but
not working in Python 3.3 as a bug.
But this is a change in behavior that [appears in the 3.3 changelog](
https://docs.python.org/3/whatsnew/3.3.html#porting-to-python-3-3).
Nowhere in the [Python Language Reference](
https://docs.python.org/3/reference/import.html#importsystem)
is there a guarantee this new behavior violates. It's an inconvenience,
but not a bug.

---

Once I found the problem I added a line in the tests to clear the cache before
each test.
I made a [pull request](https://github.com/bpython/bpython/pull/582/files)
and the maintainer Sebastian asked me to record in the code the reason for
adding this line.
An excellent idea, of course something that took several hours to understand
merits a comment! Maybe even a blog post.

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr">Tests creating modules at runtime failing in Python &gt;=3.3? importlib.invalidate_caches is there for you. Brought to you by my all morning.</p>&mdash; Thomas Ballinger (@ballingt) <a href="https://twitter.com/ballingt/status/672448020575727616">December 3, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

further reading:

* [discussion of Antoine Pitrou's patch](http://bugs.python.org/issue14043)
  introducing this behavior
* Nick Coghlan's excellent ["Traps for the Unwary in Python's Import System"](
  http://python-notes.curiousefficiency.org/en/latest/python_concepts/import_traps.html)

Thanks to Dan Luu and Allison Kaptur for comments.

[^vvv]: I went straight `python -vvvv` after `python -v`, but it turns out that
      extra V's beyond the second one don't have any effect.

[^feature]: The tests in question check that the "reimport" feature in bpython works:
      if you import a module and play with it for a few lines, modify the Python
      source code of the module you imported in a text editor,
      then hit F6 back in bpython, the new version of
      this module will be loaded and your code rerun with these modifications.
      Watch a [demo](https://asciinema.org/a/bmxvpz5r52vaf3wgwmzq6crua) or
      `pip install bpython` to give this a try.

[^source]: I was getting an import error on [line 318](
      https://github.com/bpython/bpython/blob/575cafaab9bf7f0339dd1541f6c6ce8b8885d9dd/bpython/test/test_curtsies_repl.py#L318).
      [Unittest](https://docs.python.org/3.5/library/unittest.html) test methods
      are run in an order determined by string comparison within a
      `unittest.TestCase`
      subclass, so the running `test_import_module_with_rewind` method
      was causing the `test_module_content_changed` method to fail.
