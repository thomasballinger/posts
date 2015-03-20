---
layout: post
title:  "Surprising Python"
date:   2014-03-23 21:39:00
---

Here are some things that surprise folks about Python.
I'll present one with some background, then list a few others.

--------------

Ever since [Kristen Widman built](http://www.kristenwidman.com/blog/how-to-write-a-bittorrent-client-part-1/) a [bittorrent](http://www.bittorrent.org/beps/bep_0003.html) [client](https://wiki.theory.org/BitTorrentSpecification)
at Hacker School last year, building one from scratch has been a popular project here.[^1]

A few months ago my friend Joe Wilner was having fun with the parser section of the
project, decoding a `.torrent` file. Torrent files are
[bencoded](http://en.wikipedia.org/wiki/Bencode), so the goal was to
do something like this:

    >>> bdecode("d3:dogi4e5:apple4i10ed")
    {"dog": 4, "apple": 10}

One way to write a parser (pretty similar to [Bram Cohen's
original](https://pypi.python.org/pypi/BitTorrent-bencode/5.0.8.1))
involves code like the following:

    def consume_dict(s):
        d = {}
        while s.next() != 'e':
            d[consume_string(s)] = consume_something(s)
        return d

    consume_something(iter("d3:dogi4e5:apple4i10ed"))

where `consume_string` returns a key for the dictionary being build and
consumes characters off the same iterator `s` that `consume_something`
(which dispatches to something else and may return a
dictionary, string, int, list) uses.

We couldn't figure out why the code above was broken, until he discovered

    def consume_dict(s):
        d = {}
        while s.next() != 'e':
            d.__setitem__(consume_string(s), consume_something(s))
        return d

    consume_something(iter("d3:dogi4e5:apple4i10ed"))

worked fine.

The lesson was that syntax effects the order of execution
in python dictionary assignment:
`d[second] = first`, but `d.__setitem__(first, second)`.

This is behavior that's
[well specified](http://docs.python.org/2/reference/simple_stmts.html#index-10),
but knowledge of which perhaps shouldn't be taken for granted.
Like the gain in concision of dropping the parentheses in the expression
`(a and b) or (c and d)`, maybe it's not worth the loss of clarity.

Bram's original code does this in two steps:

    def decode_dict(x, f):
        r, f = {}, f+1
        while x[f] != 'e':
            k, f = decode_string(x, f)
            r[k], f = decode_func[x[f]](x, f)
        return (r, f + 1)

which makes the order of execution quite clear.[^5]

Things I try to remember are surprising
---------------------------------------

I'm interested in knowing what pieces of Python I should
resist the temptation to use if I'm trying to write generally accessible code.
In the same way that it's _simple_ English and not English per se
that is the lingua franca of programming[^6],
I should use simple Python to clearly communicate
ideas best expressed in code to the largest audience.
This could mean not using metaclasses, but at least those
have a clear sign (the word "metaclass" in your source code)
to suggest you'll need to spend the weekend reading blog posts to 
understand the code in front of you. What it really means is not doing whatever
empirically most surprises people.

-----------

From Reid Barton's [Python quiz](http://web.archive.org/web/20101009122154/http://web.mit.edu/rwbarton/www/python.html):

    units = [1, 2]
    tens = [10, 20]
    nums = (a + b for a in units for b in tens)
    units = [3, 4]
    tens = [30, 40]
    print nums.next()

Thanks Greg Price for pointing this one out to me. A careful reading of
[the spec](http://docs.python.org/2/reference/expressions.html#generator-expressions)[^2]
yields the answer.

------------

If you come from
[Python, JavaScript](http://ballingt.com/2014/03/17/python-javascript.html),
or Ruby, this shouldn't surprise you.

    for i, x in enumerate(elements):
        if what_we_are_looking_for(x):
            break
    print i, x

If it's hard to imagine how this is surprising, know that some languages have scopes more granular
than function scope; imagine getting a new local scope every time you indented.

------------

Using in-place operators (`+=`, `*=` etc.) makes knowing whether something is
mutable (or how the author chose to implement it on a custom class) really important.

    >>> y = x = []
    >>> x = x + [1]
    >>> y
    []
    >>> y = x = []
    >>> x += [1]
    >>> y
    [1]
    >>> y = x = ()
    >>> x += (1,)
    >>> y
    ()

In-place operators are emulated if they don't exist, so `+=` could call
(`__iadd__`) or assign the left-hand side the result of `__add__`.
Even though this behavior is internally consistent and based on well-defined concepts,
any time `a += b` doesn't do the same thing as `a = a + b` you have to expect surprise.[^3]

------------

Closures: Not even once.

    >>> import functools
    >>> def foo(x):
    ...     print x
    ...
    >>> closures = [lambda: foo(i) for i in range(3)]
    >>> partials = [functools.partial(foo, i) for i in range(3)]
    >>> closures[0]()
    2
    >>> partials[0]()
    0

Yes, it's just the classic closing over the same variable, and in JavaScript you
obviously have to be familiar with this. But whenever I start to rely on
(the same) closure behavior in Python, I try
to be more Pythonic and use classes to solve the problem instead.

------------

And of course there's [the old
reliable](http://stackoverflow.com/questions/1132941/least-astonishment-in-python-the-mutable-default-argument):
mutable default arguments and
first class functions mix in interesting ways:

    >>> def foo(x=[]):
    ...     x.append(1)
    ...     return x
    ...
    >>> foo.func_defaults
    ([],)
    >>> foo()
    [1]
    >>> foo.func_defaults
    ([1],)
    >>> foo()
    [1, 1]

I try to use

    def foo(x=None):
        if x is None:
            x = []
        ...

instead,

    def foo(x=tuple(some_list)):
        ...

if I didn't really need a list in the first place, and

    def foo():
        foo.cache.append(1)
        ...
    foo.cache = []

if I really want a function with state[^4].

------------

[^1]: If you've not done much network and concurrency work before, consider
    building one!
    I think this is a great project: you get to write a parser, do http requests,
    write state machines, do raw tcp socket communication, deal with
    threads and locks or write an event-driven reactor with select, write some
    sparse data structures for representing which bytes of the file you have and
    are most rare amongst peers, write data to many files (and possibly build an
    abstraction so it feels like writing to one file or doing array assignment),
    hash pieces of data to check that they're correct, deal with various ways to
    encode information in bytes, monitor network traffic, and apply game theory a
    bit to choose your download strategy. (Or use existing libraries to do any or
    all of the above.)

[^3]: If you protest "But that's how mutable objects work in Python!" then
    check out [Ned Batchelder's terrific post on names in Python](http://nedbatchelder.com/text/names.html) for why mutability isn't the point.

[^5]: Bram's code also doesn't rely on the side effect
    of iterating on a shared iterator,
    but instead passes around an index - this more explicit
    technique also helps make it clear which read is occurring first.

[^2]: Something I'd like to get better at: I usually go straight to the source to
    see what's happening in these cases, when a trip to the spec might be
    what's really called for.
    I'm always eager to check the spec after I find something interesting
    just in case it's cPython that's wrong and not my mental model of the
    languge, and I've discovered a cool bug. Not yet :))

[^4]: And for readability, I probably _don't_ want a function with state, I
    want a class that implements `__call__` or a generator.
    Those are ways to telegraphing "State! State! I
    have icky, grubby, bug-hiding state associated with me!"

[^6]: (which has some serious privilege implications for many of us)
