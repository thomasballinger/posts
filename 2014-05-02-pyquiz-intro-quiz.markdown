---
aliases:
    - /2014/05/02/pyquiz-intro-quiz.html
    - /2014/05/02/pyquiz-intro-quiz
date: 2014-05-02T11:12:17Z
tags:
    - python
title: Python Essential Reference Tutorial Quiz
url: /pyquiz-intro-quiz/
---

Sometimes Python serves as a convenient lingua franca:
it stands in for psuedocode as a language for expressing
general programming ideas. For example,
this [excellent introduction to functional programming](http://maryrosecook.com/blog/post/a-practical-introduction-to-functional-programming) 
uses `map(function, iterable)`
to to express the same idea
that list comprehensions express.
This isn't as Pythonic, but it's more universal.
The article clearly and beautifully describes functional
programming using runnable code examples in Python that are
understandable to a wide audience.

This is great, but sometimes people writing real programs in Python
continue to think of the language as runnable psuedocode, and
I want to demonstrate that it's richer than this.
Do you know about generators? Do you understand how using indentation
for control flow really works? Set comprehensions, tuple unpacking, the
iterator protocol! 
David Beazley's
[Python Essential Reference](http://www.amazon.com/Python-Essential-Reference-4th-Edition/dp/0672329786)
is terrific for this, but I wanted something snappier.

So I present a nitpicky [quiz version of pages 5 through 25 of Python Essential Reference](/python_essential_reference_intro_quiz.html),
a brief overview of Python. It's supposed to be a learning quiz; there's no
score, just some red pixels to motivate you to understand the answer.
Some of the questions are annoying: don't worry about it,
you're not being graded. If you want to cheat (and you should cheat), consider using
[bpython-curtsies](http://ballingt.com/2013/12/21/bpython-curtsies.html)
to do so!

I'd like to create more quizzes that make the "Python is deeper than you think"
point more strongly,
improve the [quiz-writing DSL](https://github.com/thomasballinger/pythonquiz)
I wrote to write the quiz, and make
the quiz output much nicer, but wanted
to share this now in case I don't get around to those things for a while.
