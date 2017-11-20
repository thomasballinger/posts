---
date: 2017-05-06T19:00:00Z
title: Methods in Python
url: /python-methods/
---

# Meet self

In Python we have functions and methods.

Function definitions in Python look like this:

    def sloganify(x):
        return "{} or bust!".format(x)

And method definitions look like this:

    class Person:
        def sloganize(self, x):
            return "{} or bust!".format(x)

# Classes are buckets of functions

When we write Python code, we don't really write methods; we write class
statements which contain function definitions.

Class statements look like this:

    class Person:
       greeting = 'hello'
       x = 4 + 3

which is syntactic sugar for something like this:

    Person = type("Person", (), {'greeting': 'hello', 'x': 4 + 3})

So writing a class with a method

    class Foo:
        def bar(x, y, z):
            return 7

is about the same as writing

    def bar_function(x, y, z):
        return 7

    Foo = type("Foo", (), {'bar': bar_function})
    del bar_function

We can also write this as

    def bar_function(x, y, z):
        return 7

    Foo = type("Foo", (), {})
    Foo.bar = bar_function
    del bar_function

# Method objects in Python

Despite all those examples trying to show that we write functions,
not methods in Python, both function and method objects do exist:

    >>> f = Foo()
    >>> f.bar
    <bound method bar_function of <__console__.Foo object at 0x113644128>>
    >>> bar_function  # if we forgo the `del bar_function` above
    <function bar_function at 0x1135e66a8>

Besides having different names, these two different versions of the function
we wrote take different numbers of arguments!

    >>> import inspect
    >>> inspect.signature(bar_function)
    <Signature (x, y, z)>
    >>> inspect.signature(f.bar)
    <Signature (y, z)>

Is this some kind of class definition-time transformation? No, the function is indeed
what gets stored in the class!

    >>> Foo.bar
    <function bar_function at 0x1135e66a8>

Besides, the method objects we get from different instances are completely different:

    >>> f.bar
    <bound method bar_function of <__console__.Foo object at 0x113644128>>
    >>> g.bar
    <bound method Foo.bar of <__console__.Foo object at 0x10a41a710>>

Maybe the method object is created when we create an instance of the class,
and stored on the instance object!

    >>> vars(f)
    {}

Hm, I don't see it anywhere...

# Partial Application

The "one less argument" thing is familiar to people who use Python classes
and have seen this perplexing error message:

    >>> f.bar(1, 2, 3)
    Traceback (most recent call last):
      File "<input>", line 1, in <module>
        f.bar(1, 2, 3)
    TypeError: bar() takes 3 positional arguments but 4 were given

But I **did** pass three arguments! Perhaps you see where this is going.
A method is a version of a function that takes one less argument,
an example of the technique of "partial application."
Say we have the general rectangle drawing function below:

    def draw_rect(color, width, height, x, y):
        """Draws a rectangle"""
        ...

If we wanted to make a specialized version of this function for drawing small
blue square, we might write a new function that calls this old version:

    def draw_small_blue_square(x, y):
        """Draws a small blue square at the passed coordinates"""
        draw_rect('blue', 2, 2, x, y)

To be more concise and to avoid the extra layer of call stack in our error
message stack traces we could use partial application instead:

    >>> import functools
    >>> draw_large_red_square = functools.partial(draw_rect, 'red', 100, 100)

In this version we didn't write a docstring or a new signature ourselves.
`inspect.signature` is smart enough to tell how to use this function:

    >>> inspect.signature(draw_large_red_square)
    <Signature (x, y)>

but unfortunately our error message still refers to the original version of the function:

    >>> draw_large_red_square(1, 2, 3)
    Traceback (most recent call last):
      File "<input>", line 1, in <module>
        draw_large_red_square(1, 2, 3)
    TypeError: drawRect() takes 5 positional arguments but 6 were given

Partial application can be really convenient!

    >>> force_print = functools.partial(print, file=sys.stderr, flush=True)
    >>> intdict = functools.partial(collections.defaultdict, int)

Based on the similarity of these error messages, you might thing that this
method version of the function is implemented with by using
`functools.partial` to partially apply the
instance to the first parameter of the original function object. Good guess!
I looked at the CPython code and it's not. But the behavior is similar, it's sort
of a special, written-in-C, optimized version of this.

# Method binding rocks

So that's what's going on: Python is
creating a method object that takes one less argument (and now holds a reference to the
instance from which we got it) from our
our original Python function.

So method binding lets us write what could have been the clunky

    f = Foo()
    Foo.bar(f, 2, 3)

as

    f = Foo()
    f.bar(2, 3)

with both namespacing (the `f` instance looks up to its class to find
this function) and method binding (we don't have to re-specify the first
argument of `f`.

This is creation of a bound method at attribute lookup time isn't the only way
to make methods work: in JavaScript, to transformation of the function occurs
at function lookup time. To which object `this` (JavaScript's `self`) refers
depends on the syntax used to call it instead. This is often undesirable,
so the function is often transformed with `bind`.

Why is Python's behavior preferable?

    >>> distances = {'5k': 5000, 'mile': 1609.34, 'marathon': 42195}
    >>> distances.keys().sort(key=distances.get)

    >>> import threading
    >>> t = threading.thread(target=crawler.crawl)
    >>> t.start()

Because callbacks - one of the words used to describe passing a function as a value.
Sometimes Python apis expect a something callable as a passed argument.
We could wrap it in another layer of function, `.sort(lambda x: distance.get(x))`
in the first example above, but it's nice not to have to.

That pattern is more common in JavaScript:

    > setTimeout(foo.bar)  # bar won't be called correctly
    > setTimeout(function(){ foo.bar() });

# How it happens: the descriptor protocol

So method binding is cool. And a bit mysterious: somehow attribute lookup is
more complicated that we suspected, because it's somehow causing a method
to be created!
It seems like looking for the attribute
`bar` on an instance `f` would require traversing a chain like

    instance -> class -> parent class -> ... -> object

and returning the first matching attribute on one of these objects.
I remember being stunned to discover this simple model did not reflect
reality.

In fact the value returned from this process may instead
be the result of arbitrary code by implementing something known
as the descriptor protocol. Although the code for creating methods
is written in C, the power of descriptors is available to use in Python,
so let's explore it from that direction.

If an attribute implements the descriptor protocol, it will have
a `__get__()` method defined which will be called with information
about the lookup and the result returned as the result of the
attribute lookup.

    >>> class ImADescriptor():  # because I have a __get__ method
    ...     def __get__(self, instance, owner):
    ...         return 7
    ...
    >>> class Foo():
    ...     bar = ImADescriptor()
    ...
    >>> f = Foo()
    >>> f.bar
    7

The `__get__` method is called by Python infernal (whoops, internal)
attribute lookup logic in `ceval.c`
with a reference to the object that the attribute lookup started on so
it can be used to create our bound method.

Functions are implement this descriptor protocol it turns out!
Take a look at the `__get__` method on a function:

    >>> drawRect.__get__
    <method-wrapper '__get__' of function object at 0x114414268>

If we were writing out that method in Python, it might look something like

    class Function():
        def __get__(self, instance, owner):
            return functools.partial(self, instance)

---

So that's how methodization works in Python!
