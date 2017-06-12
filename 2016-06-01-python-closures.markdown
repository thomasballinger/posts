---
categories: python
date: 2016-06-01T16:00:00Z
title: Finding closure with closures
url: /python-closures/
---

This is a presentation I gave at PyCon 2016.
You can [watch the video on YouTube](https://www.youtube.com/watch?v=E9wS6LdXM8Y)
and [view the slides](https://cdn.rawgit.com/thomasballinger/python-closures-slides/master/closures.html#1)
served from the
[repo on GitHub](https://github.com/thomasballinger/python-closures-slides).

---

A friend of mine was asked what a closure was at a programming interview a few
years ago. Despite being a competent Python and JavaScript programmer who took
advantage of closures in code he wrote, he froze up at the question.
It'd be nice to have something to say in response to this question,
if not a solid definition.

Programmers more familiar with other languages have also asked me,
"Tom, you know Python; does Python even have closures?" and
"I heard Python has weak support for closures."
Once we've reached closure on this topic, I hope you'll be able to respond
productively and engage with questions and  misunderstandings about Python scope
others might have.

To find our closure we'll start with the importance of
environment to our functions and compare lexical and dynamic scope.
Then we'll follow the evolution of variables scoping in the Python language
over the last 25 years. We'll conclude that some Python certainly supports
closures, but that which Python functions count as closures and since which
version of Python they have depends on the definition of closure used.

---

Consider two functions for formatting strings: one for bolding text in
HTML, the other for bolding text in the terminal. We'll import these functions
from their respective modules using the `from ... import ... as ` syntax
because they both have the same name.

    >>> from htmlformat import bold as htmlbold
    >>> from terminalformat import bold as termbold

The source code for these two functions can be viewed with the builtin
inspect module.

    >>> import inspect
    >>> print(inspect.getsource(htmlbold))
    def bold(text):
        return '{}{}{}'.format(BOLDBEFORE, text, BOLDAFTER)

    >>> print(inspect.getsource(termbold))
    def bold(text):
        return '{}{}{}'.format(BOLDBEFORE, text, BOLDAFTER)


Although these functions appear identical, they have different behavior:

    >>> htmlbold('eggplant')
    '<b>eggplant</b>'
    >>> termbold('eggplant')
    '\x1b[1meggplant\x1b[0m'

How is this possible; what differs between these two functions?
Here's another, similar question: We saw before that the `bold`
function uses the variable `BOLDBEFORE`. Because it is neither a
parameter to the function nor a local variable, we call it a "free
variable".
If we call that function
after setting a local variable with the same name, will that
change its behavior? Will

    >>> from htmlformat import bold as htmlbold
    >>> def signbold(phrase):
    ...     BEFOREBOLD = '(in Sharpie) '
    ...     return htmlbold(phrase)
    ...
    >>> signbold('eggplant')

output `'<b>eggplant</b>'` or `'(in Sharpie) eggplant</b'`?

The question amounts to whether the Python language uses "open free
variables" whose values are determined by looking up the call
stack (dynamic scope), or closed free variables that use the value in the environment
in which the function was defined (lexical scope).

In a [1970 paper](http://www.laputan.org/pub/papers/aim-199.pdf)
describing implementions of these two approaches,
Joel Moses points out that although it might be easier to implement a language with
the first behavior, programmers are usually interested in the second.
They want their functions to use the variables they created for use with that
function, not new variables at their functions' call sites.
The answer is that Python ignores this new variable and bold tags again
surround the word eggplant.

What about changing the global variable?

    >>> from htmlformat import bold as htmlbold
    >>> BEFOREBOLD = '(in Sharpie) '
    >>> htmlbold('eggplant')

More eggplant sandwich on bold tags! The global variables in another
module are not affected by changes to global variables in this one.

Now let's finally take a look at those bold functions.

    BEFOREBOLD = '<b>'               BEFOREBOLD = '\x1b[1m'
    AFTERBOLD = '</b>'               AFTERBOLD = '\x1b[0m'

    def bold(text):                  def bold(text):
        return '{}{}{}'.format(          return '{}{}{}'.format(
          BOLDBEFORE,                      BOLDBEFORE,
          text,                            text,
          BOLDAFTER)                       BOLDAFTER)

Each formatting module has its own global variables.
Indeed, "global" variables are terribly named because there aren't global to
your whole Python program. Since we're stuck with that name, perhaps we
should imagine each module as its own planet.

When functions are imported from another module,
they emerge as emissaries from their planets with
live links back to their home worlds they use to look up variables.

For function objects in Python hold
not only a reference to the name of their home module (the `.__module__`
attribute) but also a reference to the very namespace of that module which
contains bindings from global variables names to values (`__globals__`).

In the paper mentioned earlier, Joel Moses described an implementation
of this type of behavior: for a function to behave this way, it needs both
code to execute and the environment which closes the variables use in that
function. *He called this combination of code and environment a closure*.
So Python functions are already sounding a lot like closures!

Because this is a live link, any updates to the bindings that occur in the
home module after the function is defined are still available to the function.
We can even change global bindings directly by rebinding attributes of this
data structurel that represents this environment: the imported module object.

---

The distinction between function definition time and function execution
becomes important with this "live link" behavior.
It turns out that Python analyzes function source code,
even compiles it, when a function is defined.
During this process it determines the scope of each variable. This determines
the process that will be used to find the value of each variables,
but does not actually look up this value yet.

A Python function object is the result of this process. Each of its
attributes stores a different piece of computer-readable information about
about the function. Most of this information is in the code object stored
by the `__code__` attribute.

Since Python has been available on the internet, there have been at least two
types of variables in functions: local variables and global variables.
Local variables (including function parameters) appear in `.__code__.co_varnames`
and global variables and a few other things make up `.__code__.co_names`.
Identifying the scope of a variable is a task Python programmers do
frequently as they read code, so you may already have an intuition for
the rules. Let's try at a few examples to understand the rule.

    >>> def movie_titleize(phrase):
    ...     capitalized = phrase.title()
    ...     return capitalized + ": The Untold Story"

In this function for building great movie titles, are `phrase` and
`capitalized` local or global variables?

They are both local. One is a parameter to the function, the other is
assigned to on the first line. This type of function, sometimes called
a "pure" function, doesn't need its link to its home module for looking up
variables. Without this associated environment, the function would not be a
closure, and here we find out first fork in the meaning of the word. Is
a function a closure if it has this link to its defining environment but
that environment is never used? Some would say functions require free
variables to be closures, others that the combination of code and environment
is enough, so long as the Python doesn't remove this link
link to home module. For an altogether different reason,
most would say that none of the functions we have seen so far are closures.
Hang on for that reason in a few minutes.

    >>> def catchy(phrase):
    ...     options = [phrase.title(), DEFAULT_TITLE]
    ...     options.sort(key=catchiness)
    ...     return options[1]
    ...

In this function for finding catchy phrases, are `phrase`, `options`,
`DEFAULT_TITLE`, and `catchiness` local variables or global variables?
Once you decide, you can find out whether you agree with the Python
interpreter by checking those interesting attributes of the function's
code object:

    >>> catchy.__code__.co_varnames
    ('phrase', 'options')
    >>> catchy.__code__.co_names
    ('catchiness', 'DEFAULT_TITLE', 'sort', 'catchiness')

`phrase` and `options` are local variables because the first was a parameter
and the second was assigned to. `DEFAULT_TITLE` and `catchiness` fit neither
of these descriptions so they are global variables.
A few extra strings are in the `co_names` tuple because Python uses this
list for most that storing global variables.
If you saw this function in some source code and wanted to copy it alone to use,
you wouldn't be able to: there's important environment information you
would also need to include.

Your programmer intuition might disagree with Python's categorization
in this next example.

    >>> HIGH_SCORE = 1000
    >>> def new_high_score(score):
    ...     print('congrats on the high score!')
    ...     print('old high score:', HIGH_SCORE)
    ...     HIGH_SCORE = score
    ... 
    >>> new_high_score(1042)

It certainly looks like the author of the function wanted `HIGH_SCORE` to
be a global variable, but Python categorizes it as a local variable
because it's assigned to in the function. Calling the function results
in an UnboundLocalError because the variable, considered local for the
entirety of the function, doesn't have a value assigned yet when it's
printed as the old high score.

The programmer can express this authorial intent to Python with the `global`
keyword, which changes the categorization of `HIGH_SCORE` from local variable
to global.

    >>> HIGH_SCORE = 1000
    >>> def new_high_score(score):
    ...     global HIGH_SCORE
    ...     print('congrats on the high score!')
    ...     print('old high score:', HIGH_SCORE)
    ...     HIGH_SCORE = score
    ...
    >>> new_high_score.__code__.co_varnames
    ('score', 'HIGH_SCORE')
    >>> new_high_score.__code__.co_names
    ('print',)
    >>> new_high_score(1042)
    congrats on the high score!
    old high score: 1000

With the global keyword we've now completed a description of how scope
has worked in Python from its inception through to Python 2.0 in the year 2000.
Python functions have always closed over their own module-level environment.
But an important method of closing free variables was still not available to
us, one required by most to classify a function as a closure:
using outer scopes that are not the global scope.

    def tallest_building():
        buildings = {'Burj Khalifa': 828,
                     'Shanghai Tower': 632,
                     'Abraj Al-Bait': 601}

        def height(name):
            return buildings[name]

        return max(buildings.keys(), key=height)

Are the variables `name` and `buildings` local or global variables in the
`height` function above? `name` is certainly local as a parameter, but
`buildings` is neither local or global, it comes from an outer non-global
scope. Since `buildings` is not a local variable it is assumed to be global
in Python 2.0 and calling it produces "global name 'buildings' is
not defined" NameError. Optionally in Python 2.1, then by default in Python
2.2, variables from outer non-global scopes were added and are found at
`.__code__.co_freevars`:

    >>> height.__code__.co_varnames
    ('name',)
    >>> height.__code__.co_names
    ()
    >>> height.__code__.co_freevars
    ('buildings',)

Typically when people talk about closures they mean closing around these
in between outer scopes that are neither local nor global. Closing over
the module-level "global" scope is considered a special case, and indeed
is simpler to implement. You may already be familiar with module
objects in Python: generally they're singletons, so a given module has only
one mapping of variables to values. But a function can be run many times,
producing many different mappings of its local variables to values.
Each of which must be kept track of so long as a function that was defined
in this or an enclosing scope still exists.

    formatters = {}
    colors = ['red', 'green', 'blue']
    for color in colors:
        def in_color(s):
            return ('<span style="color:' +
                    color + '">' + s + '</span>')
        formatters[color] = in_color


    formatters['green']('hello')

The code above defines several functions for formatting text in color in html.
With which color does the green one of these functions format the text
`'hello'`?
Since these three functions were defined in the same environment, they share
the same mapping of variables to values. If we consider the value of the
`color` variable once the for loop has finished, it becomes clear that all
three functions have the same behavior: coloring strings blue. If each
function is to have a different value associated with the color variable it is
necessary to create separate scopes for these functions to be defined in:

    formatters = {}
    colors = ['red', 'green', 'blue']
    def make_color_func(color):
        def in_color(s):
            return ('<span style="color:' +
                    color + '">' + s + '</span>')
        return in_color

    for color in colors:
        formatters[color] = make_color_func(color)

    formatters['green']('hello')

Each time the `make_color_func` function is called, a new local mapping
is created binding color to one of red, green or blue; a function called
`in_color` is defined which references the color variable in this outer scope;
and the `in_color` function is returned and stuck in a dictionary.

This solution to the "late-binding" behavior of Python relies on separate
scopes being created for each function and requires that Python maintain
three sets of bindings for the `make_color_func` function's local scope.
Precisely how these bindings are maintained by Python is beyond our scope here,
but the `.__closure__` attribute on each of the three produced functions provides
some hint.

We've reached the most common definition of a closure: a function with
variables closed by an outer, non-global scope.
However another fork in definitions occurs here: some would call our three
color functions closures and but not the earlier height function because it was used
in the same scope it was defined. Although CPython doesn't implement the two any
differently, you can imagine that it becomes more difficult to maintain the
environment a function to evaluate its variables once the bindings it needs
goes out of scope. The distinction here is that looking up the stack instead
of the "closure" solution of code + environment would result in the same
behavior in the first case, making whether a function was a closure or not
only distinguishable in the second case.

So by Python 2.2, functions in Python are definitely closures: every function
is always a closure by my most library definition
since they all carry environment with them,
and only those which reference variables which have gone out of scope by the
strictest. Nothing much changes with scope through the Python 2.x series, so
we have now covered scope in Python up through 2.7, through to the year 2008.

But if rumblings of the insufficiency of Python's closures have ever
reached your ears, you may not have found your closure yet. You may have
heard that Python has "weak support" for closures,
or that Python has "read-only" closures, not "full" closures.
This comes from an asymmetry between global variables and outer
non-global variables, which I will from now on refer to as "nonlocal" variables.

    >>> def get_number_guesser(answer):
    ...     last_guess = None
    ...     def guess(n):
    ...         if n == last_guess:
    ...             print('already guessed that!')
    ...         last_guess = n
    ...         return n == answer
    ... 
    >>> guess = get_number_guesser(12)
    >>> guess(9)

Like the earlier example demonstrating the usefulness of the `global` keyword,
the inner `guess` function above assigns to the variable `last_guess` that the
programmer meant not to be local. How can Python be informed of this intent?
With the new `nonlocal` keyword in Python 3.

Without `nonlocal`, nonlocal variables cannot be rebound to new values.
Nonlocal mutable objects can be mutated for a similar effect, but the identity
of the object in an outer binding cannot be changed. But since Python 3,
we definitely have "full" closures now; there are no more missing details.

As with the global keyword the change in semantics may seem small, but its
lack is met with incredulity in Python 2 by some familiar with closures in
other languages. As we find our closure with what closures are and whether
they exist in Python, a new question arises: how did we get on without them
for so long?

---

We use closures all over the place in Python: inner functions
(often written with the `lambda` syntax)
that reference outer scopes abound, and the use of functions
in interfaces as callbacks makes their use more likely.
Decorators always take a function as an argument and often define a new
function to replace it, which itself typically holds a reference to the old
function through a free variable from the outer function scope of the
decorator. We can inspect a function for its `.__closure__` attribute
to see if it contains free variables that are closed by outer, nonlocal
scopes for those who demand this of their closures.

Adding the nonlocal keyword took nine years, from Python 2.2 in 2001 to Python 3 in 2008.
If it's so important a change, why don't we see a ton of code using it now?

The `global` keyword may have delayed this need: module-level bindings
have been modifiable in Python for a long time. And now that nonlocal
is here, the need for compatibility with Python 2 code that many library authors
have prevents some uses. Consider this abridged excerpt from Django:

    def decorating_function(user_function):
        ...
        nonlocal_root = [root]  # make updateable non-locally

        def wrapper():
            nonlocal_root[0] = oldroot[NEXT]
            ...

Since the root variable in the outer function cannot directly changed, it is
stuck in the simplest possible mutable object -- a list -- which is mutated
to imitate rebinding. Based on the name, it's clear both that nonlocal would
be a good fit here and that the author of this code knew that when they wrote
it. But compatibility with Python 2 forces the word nonlocal to be used here
only to evoke the idea of a nonlocal variable.

This pattern is less common than in some languages because of Python's
excellent object system, in particular its ability to bind methods to objects.
When a callback function which accesses or modifies some internal state is
needed, often that state will be placed in a class instance. State on objects
in Python is readable and instrospectable, and the methods of an object
can be used as callbacks.

    def tallest_building():
        buildings = {'Burj Khalifa': 828,
                     'Shanghai Tower': 632,
                     'Abraj Al-Bait': 601}

        return max(buildings.keys(), key=buildings.get)

Here the `get` method of the builtin Python dictionary object is used as a
callback, which concisely expresses what data the method will operate on.

And finally I posit a cultural reason: Python programmers tend to be
comfortable with private data being externally accessible. Python and
JavaScript are relatively similar languages, and both lack (or at least in certain
versions have lacked) private object data which can be accessed by methods
of the object but not by outside code, instead using conventions like a single
underscore to inform users that an attribute is not part of the public
interface with that object. In both languages the following
pattern is possible, but in JavaScript it is commonplace while in Python it is
unheard of.

    >>> class Person(object): pass
    >>> def create_person(name):
    ...     age = 10
    ...     p = Person()
    ...     def birthday():
    ...         nonlocal age
    ...         age = age + 1
    ...     p.birthday = birthday
    ...     p.greet = lambda: print("Hi, I'm", name, "and I'm", age, "years old")
    ...     return p
    ... 
    >>> me = create_person('Tom')
    >>> me.birthday()
    >>> me.age
    Traceback (most recent call last):
      File "<input>", line 1, in <module>
    AttributeError: 'Problem' object has no attribute 'age'

The above code hides data in local variables of a constructor function
which inner functions have access to, then adds these methods to the `Person`
instance. Now the `Person` instance has methods for accessing and modifying private
data that are not attributes of the object itself. Again, this pattern is entirely
possible and achieves the same aim as in JavaScript, but culturally isn't used
in Python.

---

I think it's fine that we don't use rebinding closures all that much.
In new code nonlocal should be used when appropriate instead of the mutable
object hack we saw above, but it's fine for it to remain relatively rare.

Some modern Python functions are most certainly closures: whether it's all of them
or just a few depends on the definition. Is it enough to be capable of
referring to variables from outer scopes (all Python functions), or must the
functions make use of this ability? Must these outer scopes not be global?
Must the scopes referenced by the free variables go out of scope to prove a
function is a closure, or is storing the environment such that the variables
could go out of scope enough? And must closures be able to rebind these
free variables, disqualifying all Python 2 functions?

I like the "all Python 2.2 and greater functions which close over outer,
non-global scopes are closures" answer, but
have found closure in knowing the discussion to have if I were asked.

I hope I've helped you find closure with closures.

---

Further reading:

* [An earlier post about rebinding closures in Python](http://ballingt.com/rebinding-closures-in-python)
* Ned Batchelder's [Facts and myths about Python names and values](http://nedbatchelder.com/text/names.html) and later [PyCon talk](http://nedbatchelder.com/text/names1.html)
* [PEP 227 Statically Nested Scopes](https://www.python.org/dev/peps/pep-0227/), which includes notes on closure implementation
* [PEP 3104 Access to Names in Outer Scopes](https://www.python.org/dev/peps/pep-3104/)
* Wikipedia: [Closure (computer programming)](https://en.wikipedia.org/wiki/Closure_(computer_programming))

Related Python topics:

* builtins: last resort of failed global variable lookups
* `__closure__` and `__code__.cell_vars`: how closures are implemented
* bytecode: what does "compiling" a function really mean?
* descriptors and method binding: the dark secret that turns functions into methods
* scopes of various comprehensions and generator expressions: I lied when I
  said scope hasn't changed much

Others' thoughts on closures in Python:

* effbot: [Closures in Python](http://effbot.org/zone/closure.htm)
* Stack Overflow: [Why aren't Python nested functions called closures?](http://stackoverflow.com/questions/4020419/why-arent-python-nested-functions-called-closures)
* ynniv: [Closures in Python](http://ynniv.com/blog/2007/08/closures-in-python.html)
* Python Tutorial: [Python Scopes and Namespaces](https://docs.python.org/3.5/tutorial/classes.html#python-scopes-and-namespaces)

Others' thoughts on closures:

* Joel Moses: [The Function of FUNCTION in LISP](http://www.laputan.org/pub/papers/aim-199.pdf)
* Martin Fowloer: [Lambda](http://martinfowler.com/bliki/Lambda.html)
* MDN: [JavaScript Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)
* Stack Overflow: [What is a closure?](http://stackoverflow.com/questions/36636/what-is-a-closure)
* programmers.stackexchange: [What is a closure?](http://programmers.stackexchange.com/questions/40454/what-is-a-closure)

