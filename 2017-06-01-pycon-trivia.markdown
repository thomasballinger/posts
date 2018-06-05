---
date: 2017-06-01T10:00:00Z
title: PyCon Trivia 2017
url: /pycon-trivia/
---

Brandon Rhodes, in addition to chairing the PyCon and giving a talk this year,
MC’d the Python trivia night as he has done for the past three years.
([2014](http://rhodesmill.org/brandon/2014/pycon-trivia-night/),
[2015](http://rhodesmill.org/brandon/2015/pycon-trivia-night/),
[2016](http://rhodesmill.org/brandon/2016/pycon-trivia-night/))

This year for some reason (maybe because I see him [at
work](https://blogs.dropbox.com/tech/)) Brandon asked me to help out with the
trivia night. Along with Larry Hastings, each of the three of us wrote eight
trivia questions.  I wrote questions for the second section of trivia of night.

I tried to come up with questions I imagined might incite discussion and participation,
but still reward folks who had been using Python for many years.

# Question 1
Which of these is not, and never has been, a real CPython error message?

* a. “name foo is not defined”
* b. “local variable foo referenced before assignment”
* c. “can not delete variable foo referenced in nested scope”
* d. “cell variable foo referenced in enclosing scope after deletion”
* e. “free variable foo referenced before assignment in enclosing scope”

The answer is that d, “cell variable foo referenced in enclosing scope after deletion” has never been a CPython error message. Here’s an interactive Python 3 session that produces errors a, b, and e:

{{< highlight python >}}
>>> foo
Traceback (most recent call last):
  File "<input>", line 1, in <module>
    foo
NameError: name 'foo' is not defined
>>> def function():
...     foo
...     foo = 1
...
>>> function()
Traceback (most recent call last):
  File "<input>", line 1, in <module>
    function()
  File "<input>", line 2, in function
    foo
UnboundLocalError: local variable 'foo' referenced before assignment
>>> def function():
...     def inner():
...         foo
...     inner()
...     foo = 1
...
>>> function()
Traceback (most recent call last):
  File "<input>", line 1, in <module>
    function()
  File "<input>", line 4, in function
    inner()
  File "<input>", line 3, in inner
    foo
NameError: free variable 'foo' referenced before assignment in enclosing scope
{{< / highlight >}}

Error c only exists in Python 2:

{{< highlight python >}}
>>> def function():
...     foo = 1
...     def inner():
...         foo
...     del foo
SyntaxError: can not delete variable 'foo' referenced in nested scope
>>>
{{< / highlight >}}

and when we try the same code in Python 3 (or what we were planning to type before that pesky syntax error stopped us from typing the next like of our function)

{{< highlight python >}}
>>> def function():
...     foo = 1
...     def inner():
...         foo
...     del foo
...     inner()
...
>>> function()
Traceback (most recent call last):
  File "<input>", line 1, in <module>
    function()
  File "<input>", line 6, in function
    inner()
  File "<input>", line 4, in inner
    foo
NameError: free variable 'foo' referenced before assignment in enclosing scope
we get a runtime error instead.
{{< / highlight >}}

What does error d, “cell variable foo referenced in enclosing scope after deletion” mean? I made it up, imagining Python was trying to give a more descriptive message describing this last case.

# Question 2

We made some last minute changes to question 2, and I don’t have record of the final version. It something like:

Which of these features were not added until Python 3?

* a. cpython prompt gets readline completion by default
* b. nested scopes
* c. list comprehensions
* d. generator expressions

The answer is that list comprehensions (2.0), generator expressions (2.4) and nested scopes (2.2, really 2.1 in the first __future__ compiler directive) are all from Python 2, while CPython did not get readline completion by default, requiring

{{< highlight python >}}
>>> import rlcompleter
>>> import readline
>>> readline.parse_and_bind("tab: complete")
until Python 3.4.
{{< / highlight >}}

I had originally wanted to require teams to arrange these features in the order in which they were added to Python, planning to score answers with one minus the Kendall tau distance of answer to the key, aka the number of bubblesort swaps required. But Brandon helpfully pointed out this would be difficult to grade on the spot, so we changed the question to be 5 binary 2 or 3 answers.

# Question 3
In what order do the function a, b, and c run in?

{{< highlight python >}}
a()[b()] = c()
{{< / highlight >}}

The answer is c, then a, then b. This bit me when someone was writing a
bencode parser, see [this post](http://ballingt.com/surprising-python) for the story. I was surprised that most teams got this.

Answer: c, a, b

# Question 4
What’s the longest Python keyword?

It’s a tie between “continue” and “nonlocal.” I hoped this would be a fun, if easy question, because it’s the kind of thing no one knows off the top of their head. Not a single team missed this question, but I hope it was fun to brainstorm keywords.

# Question 5
Given a class “A” with a method “b” and an instance “a”, what is the type of: (in Python 2 or 3) 

* `a.b`
* `A.b`

This question come almost directly from
[the talk](http://ballingt.com/surprising-python)
I [gave at PyCon](https://www.youtube.com/watch?v=byff9LhYXOg)
this year, which I think wasn’t unfair because it was not until the following day.
The answers are:

* a.b: py2: instancemethod, py3: method
* A.b: py2: instancemethod, py3: function

# Question 6
Many of the most fundamental types in Python are available as builtins, but some must be obtained from the types module. Name 3 types available in the types module.

This was a tricky question to grade: the answers I wanted were things like:

* `async_generator`
* `builtin_function_or_method`
* `code`
* `coroutine`
* `frame``
* `function`
* `generator`
* `getset_descriptor`
* `mappingproxy`
* `member_descriptor`
* `method`
* `module`
* `traceback`
* `DynamicClassAttribute`
* `SimpleNamespace`
* `_GeneratorWrapper`

because this is what the types are actually called. But in the types module most of these types are bound by different names:

* `AsyncGeneratorType`
* `BuiltinFunctionType`
* `BuiltinMethodType`
* `CodeType`
* `CoroutineType`
* `DynamicClassAttribute`
* `FrameType`
* `FunctionType`
* `GeneratorType`
* `GetSetDescriptorType`
* `LambdaType`
* `MappingProxyType`
* `MemberDescriptorType`
* `MethodType`
* `ModuleType`

I ended up accepting both.

# Question 7
In addition to modules, keywords, and builtins, typing `help()` in the interactive intepreter lets you ask about topics! Some examples: ASSERTION ASSIGNMENT ATTRIBUTES

Name three more of these topics.

There are a bunch. It was clear some teams were guessing, but many guesses were correct. Then a couple teams used really obscure ones to show they really knew these. Maybe they had a team member that had written some of these help messages!

To see a full list of these:

{{< highlight python >}}
>>> help()
>>> topics
{{< / highlight >}}

# Question 8

“If the implementation is `__________`, it’s a bad idea.”

Answer: hard to explain

