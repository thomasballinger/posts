---
aliases:
    - /2014/03/01/nonblocking-stdin-in-python-3.html
    - /2014/03/01/nonblocking-stdin-in-python-3
date: 2014-03-01T23:10:00Z
tags:
    - python
    - terminals
title: Nonblocking stdin read works differently in Python 3
url: /nonblocking-stdin-in-python-3/
---

I added better paste support to
[bpython-curtsies](http://ballingt.com/2013/12/21/bpython-curtsies.html) this week, and when I
finally got around to testing the feature in Python 3 I found things weren't
working as expected. The bug was to do with doing nonblocking
reads of stdin working differently in Python 2 vs 3.

Once I traced the problem (pressing keys didn't do anything) to recent changes
I'd made to the [terminal wrapper library](https://github.com/thomasballinger/curtsies),
used by the project, I ran its tests. I wasn't particularly hopeful this
would solve the problem, and I wasn't surprised when they all passed.
The fiddly string manipulation bits of curtsies are
well tested, but terminal interaction and stream reading aren't at all.

Here's my minimal example of a nonblocking read of stdin in Python 2:
(this is worse than this [effbot](http://effbot.org/pyfaq/how-do-i-get-a-single-keypress-at-a-time.htm) example (which
helpfully pointed me to fcntl when I originally wrote the code) because it
doesn't use a select, but I've
found stdin sometimes doesn't get returned from a select call for a third of a second or so
after I read a byte on it, despite there being more bytes lined up ready to be
read.)

    import fcntl
    import sys
    import os
    import time
    import tty
    import termios

    class raw(object):
        def __init__(self, stream):
            self.stream = stream
            self.fd = self.stream.fileno()
        def __enter__(self):
            self.original_stty = termios.tcgetattr(self.stream)
            tty.setcbreak(self.stream)
        def __exit__(self, type, value, traceback):
            termios.tcsetattr(self.stream, termios.TCSANOW, self.original_stty)

    class nonblocking(object):
        def __init__(self, stream):
            self.stream = stream
            self.fd = self.stream.fileno()
        def __enter__(self):
            self.orig_fl = fcntl.fcntl(self.fd, fcntl.F_GETFL)
            fcntl.fcntl(self.fd, fcntl.F_SETFL, self.orig_fl | os.O_NONBLOCK)
        def __exit__(self, *args):
            fcntl.fcntl(self.fd, fcntl.F_SETFL, self.orig_fl)

    with raw(sys.stdin):
        with nonblocking(sys.stdin):
            while True:
                try:
                    c = sys.stdin.read(1)
                    print(repr(c))
                except IOError:
                    print('not ready')
                time.sleep(.1)

I've split up the context managers like this because that's the way it works
in [curtsies](https://github.com/thomasballinger/curtsies): we're always in
cbreak mode to receive keypresses immediately, but we occasionally go into
nonblocking mode to check to see if there's another keypress already ready to
be read; if there is, we assume the user has pasted text into our stream.

In Python 2.7, the above code works fine, but never raises the IOError in
Python3. After googling a bit I found
David Beazley's old presentation on [Python 3 I/O](http://www.slideshare.net/dabeaz/mastering-python-3-io-version-2).
No specific detail in it was particularly useful to me (though I did learn
lots of generally useful things, chief among them that the [array
module](http://docs.python.org/2/library/array.html) exists) besides that
IO had completely been redone for Python 3 -- so I needed to expect the
unexpected.

I went back to my example above, where the read method kept
returning an empty string when it should be erroring on having no bytes to
read, and it
occurred to me that this must be the new interface: nonblocking files return
an empty string on `.read(1)` if there's nothing to read! While this feels
inconsistent with nonblocking socket behavior, it's convenient here; there's
no more IOError to catch.

I haven't seen docs that specify this behavior yet, but things seem to be
working alright on my platform.

    with raw(sys.stdin):
        with nonblocking(sys.stdin):
            while True:
                c = sys.stdin.read(1)
                if c:
                    print(repr(c))
                else:
                    print('not ready')
                time.sleep(.1)
