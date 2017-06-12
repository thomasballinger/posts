---
categories: python javascript
date: 2016-01-05T12:00:00Z
title: Rebinding closures in Python
url: /rebinding-closures-in-python/
---

[Since the release of Python 2.2](
https://docs.python.org/3/whatsnew/2.2.html#pep-227-nested-scopes)
in 2001, all Python functions have closed over bindings in outer scopes.
Before this release variables referenced from outer scopes had to be passed in
as arguments. A common workaround to this limiting behavior
was to set these at function definition time
with keyword arguments:

{{< highlight python >}}
#!/usr/bin/env python2.0
def find(self, name):
    return filter(lambda x, name=name: x == name, self.list_attribute)
{{< / highlight >}}

Since 2001 this workaround is no longer necessary and the references
to `name` in the body of the lambda function are resolved using the
current value in the outer `find` function:

{{< highlight python >}}
#!/usr/bin/env python2.2
def find(self, name):
    return filter(lambda x: x == name, self.list_attribute)
{{< / highlight >}}

However Python functions could not *modify* these bindings:
although an object referred to by a variable from an outer
scope can be modified, like appending an item to a list,

{{< highlight python >}}
#!/usr/bin/env python2
def get_odds(candidates):
    odds = []
    def add_if_odd(x):
        if x % 2 == 1:
          l.append(x)  # This type of modification is fine.
    for x in candidates:
        add_if_odd(x)
    return odds
{{< / highlight >}}

a function could not change
to which object a variable from an outer scope referred.

{{< highlight python >}}
#!/usr/bin/env python2
def largest_odd(candidates):
    highest = 1
    def replace_if_odd(x):
        if x % 2 == 1:
           highest = x  # Reassigning highest does not work!
                        # This creates a local variable instead.
    for x in sorted(candidates):
        save_if_odd(x)
    return highest
{{< / highlight >}}

One of my favorite changes in Python 3 is the addition of the nonlocal
keyword. Described by [PEP 3104](https://www.python.org/dev/peps/pep-3104/)[^PEPs]
as "access to names in outer scopes" and in other places as "rebinding
closures" and "read/write closures," this allows functions to rebind variables
in outer scopes. By marking a variable name nonlocal we specify
that when that variable name is assigned to in this function,
we mean the already existing binding from outside this function.
The nonlocal keyword works analogously to the global keyword which
allows reassignment of global variables in Python 2 and 3.

{{< highlight python >}}
#!/usr/bin/env python3
def largest_odd(candidates):
    highest = 1
    def replace_if_odd(x):
        nonlocal highest  # Nonlocal only exists in python 3.
        if x % 2 == 1:  # Because of the previous line
           highest = x  # this rebinds the outer highest.
    for x in sorted(candidates):
        save_if_odd(x)
    return highest
{{< / highlight >}}

I think this is a great addition that helps me write code the way I would
in JavaScript or a Lisp. But despite this lack of rebinding closures in Python
2 frustrating some programmers coming from these and other languages,
it doesn't seem to be a big deal to Python programmers.
The feature doesn't make the top 11 of Aaron Meurer's
[top 10 awesome features of Python that you can't use because you refuse to upgrade to Python 3](
https://asmeurer.github.io/python3-presentation/slides.html).

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr">Folks who write Python code, what do you think of the new nonlocal keyword in Python 3?</p>&mdash; Thomas Ballinger (@ballingt) <a href="https://twitter.com/ballingt/status/684446320279597056">January 5, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>
Here are some thoughts on why this situation doesn't come up
very much in Python and how people worked around not having
it in Python 2.

# Global scope isn't

Python 2 functions can rebind some variables:
the module-level variables known as globals.
`global`, available in Python 2, marks a variable name
in a function as referring to the global variable in that module.
This provides some fraction of the power of full closures
since every Python function closes over the "global" variables
of the module in which it was defined.

I also find I write less local functions in Python because of
the built-in module system than I do in JavaScript, and these
top-level functions have no enclosing scope besides the global
scope to use `nonlocal` on.

# Community norm: no data hiding

An emblematic use of closures in JavaScript
is the protection of private data,
but in Python data privacy isn't enforced.

While using a closure to hide data is a
common pattern in JavaScript,

{{< highlight javascript >}}
function createPerson(name){
    var age = 10;
    return {
        birthday : function(){
            age = age + 1;
        },
        greet : function(){
            console.log("Hi, I'm", name, "and I'm", age, "years old");
        }
    }
}

var me = createPerson('Tom')
me.greet()
{{< / highlight >}}

that same pattern in Python 3 would be
considered pretty weird code:

{{< highlight python >}}
#!/usr/bin/env python3
class Person(object):
    pass

    @staticmethod
    def create(name):
        age = 10
        p = Person()
        def birthday():
            nonlocal age
            age = age + 1

        def greet():
            print("Hi, I'm", name, "and I'm", age, "years old")

        p.birthday = birthday
        p.greet = greet
        return p

me = Person.create('Tom')
me.greet()

{{< / highlight >}}

# Mutable object hack

Writing to a closed over variable is useful for producing a callback
which records somewhere that it was called.
Here's a signal handing example in Python 3:

{{< highlight python >}}
#!/usr/bin/env python3
from signal import signal, SIGWINCH

def notice_window():
    window_size_changed = False

    def handler(signum, frame):
        nonlocal window_size_changed
        window_size_changed = True

    signal(SIGWINCH, handler)

    while True:
        if window_size_changed:
            window_size_changed = False
            print('window size has changed')
{{< / highlight >}}

The most direct translation of the above is to use
the simplest possible mutable object to store
the boolean `window_size_changed` value.
Read-only closures with mutable objects seem
to provide an (ugly) equivalent to full closures:

{{< highlight python >}}
#!/usr/bin/env python2
from signal import signal, SIGWINCH

def notice_window_with_mutable_object():
    window_size_changed = [False]

    def handler(signum, frame):
        window_size_changed[0] = True

    signal(SIGWINCH, handler)

    while True:
        if window_size_changed[0]:
            window_size_changed[0] = False
            print('window size has changed')
{{< / highlight >}}

This works in every case I can think of, but is ugly because
the list only obfuscates what's going on.
When I see this pattern in Python 2 code I wish it were Python 3
so it could be fixed, but it seems pretty rare.

# Mutability and method binding:

It's much more common to find a function that could rebind an outer variable,
but instead mutates an object more descriptive than a one-element list.

When passing in a callback to a function a Python method is
particularly convenient because that method will be bound to that object
at attribute lookup time. This more clearly communicates what state
might be modified because methods of objects often change the state of the
objects to which the belong.

{{< highlight python >}}
#!/usr/bin/env python2
class Noticer:
    def __init__(self):
        self.window_size_changed = False

    def handler(self, signum, frame):
        self.window_size_changed = True

def notice_window_with_method():
    noticer = Noticer()
    signal(SIGWINCH, noticer.handler)
    while True:
        if noticer.window_size_changed:
            noticer.window_size_changed = False
            print('window size has changed')
{{< / highlight >}}

# Less callback-oriented interfaces

Python codebases seem to prefer passing objects that conform to informal
interfaces to passing functions directly, using attributes of that object
for the state being changed.

Further reading:

* [PEP 3104: Access to Names in Outer
  Scopes](https://www.python.org/dev/peps/pep-3104/)
* [PEP 227: Statically Nested
  Scopes](https://www.python.org/dev/peps/pep-0227/)
* Stack Overflow posts about workarounds for Python 2: [1](http://stackoverflow.com/questions/21959985/why-cant-python-increment-variable-in-closure)
  [2](http://stackoverflow.com/questions/2009402/read-write-python-closures)
  [3](http://stackoverflow.com/questions/392349/modify-bound-variables-of-a-closure-in-python)
  [4](http://stackoverflow.com/questions/12091973/python-closure-with-assigning-outer-variable-inside-inner-function)
  [5](http://stackoverflow.com/questions/8447947/is-it-possible-to-modify-variable-in-python-that-is-in-outer-but-not-global-sc)

Thanks Julia Evans and Lindsey Kuper for comments and corrections.

[^PEPs]: Python enhancement proposals are great reading! They summarize
         mailing list discussion by a lot of smart people with Python
         experience considering every angle of a new feature. I recently
         listened to [a `Podcast.__init__` episode](
         http://podcastinit.com/the-pep-talk.html) about PEPs that was
         fun and informative too.
