---
date: 2018-04-06T19:00:00Z
title: "Python is not Java or C++: Python as a second language empathy"
url: /python-second-language-empathy/
---

<style>
img {
  width: 100%;
}
ul {
  margin: 1em 0;
}
</style>

[abstract](https://www.pycascades.com/talks/python-is-not-java-or-c/)

{{< youtube id="kWNBK2cnaYE" >}}

---

# Python is not Java, nor is it C++

It's different! Let's talk about how.

# Why?

Because as Python experts (you did choose to come to a Python conference so likely you're either an expert already or in time you're going to be come one if you keep going to Python conferences) we have a responsibility to help our colleagues and collaborators who don't know Python as well we do. The part of that responsibility I want to focus on today is when other people have experience with other programming languages but are new to Python.

I work at Dropbox, which as Guido said earlier today is a company that uses a fair bit of Python. But a lot of programmers come to Dropbox without having significant Python experience.  Do these people take a few months off when they join to really focus on learning and figure out exactly how Python works, having a lot of fun while they do it? That would great (*briefly shows slide of Recurse Center logo*) but that's not what usually happens. Instead they learn on the job, they start making progress right away. They'll read some books (my favorite is [Python Essential Reference](http://www.dabeaz.com/per.html), but I hear [Fluent Python](http://shop.oreilly.com/product/0636920032519.do) is terrific), watch some [Python talks](https://www.youtube.com/c/PyCascades), read some [blog posts](https://nedbatchelder.com/), ask questions at work, and Google a lot. That last one is the main one, lots of Google and lots of Stack Overflow.

Learning primarily by Googling can leave you with certain blind spots. If the way that you're learning a language is by looking up things that are confusing to you, things that aren't obviously confusing aren't going to come up.

We ought to be trying to understand our colleagues' understandings of Python. This is a big thing whenever you're teaching, whenever you're trying to communicate with another person: trying to figure out their mental model of a situation and providing just the right conceptual stepping stones to update that model to a more useful state.

# Python-as-a-secondary-language empathy

We should try to understand the understandings of Python of people coming to Python as a new language. I'm going to call this "Python-as-a-second-language empathy."

How do we build this PaaSL empathy thing?

The best thing you can do is learn another language first, and then learn Python. Who here has another language that they knew pretty well before learning Python? (most hands go up) Great! Terrific! That's a superpower you have that I can never have. I can never unlearn Python, become fluent in another language, and then learn Python again. You have this perspective that I can't have. I encourage you to use that superpower to help others with backgrounds similar to your own. I'd love to see "Django for Salesforce Programmers" as a talk at a Python conference because it's very efficient when teaching to be able to make connections to a shared existing knowledge base.

Another thing you can do to build this PAASL empathy (I'm still deciding on an acronym) is to learn language that are different than the ones you know. Every time you learn a new language you're learning new dimensions on which someone could have a misconception.

Consider the following:

{{< highlight python >}}
a = b + c
{{< /highlight >}}

Depending on the languages you know, you might make different assumptions about the answers to the following questions:

* Will a always be equivalent to the sum of b and c from now on, or will that only be true right after we run this code?
* Will b + c be evaluated right now, or when a is used later?
* Could b and c be function calls with side effects?
* Which will be evaluated first?
* What does plus mean, and how do we find out?
* Is `a` a new variable, and if so is it global now?
* Does the value stored in `a` know the name of that variable?

These are questions you can have and ways that someone might be confused, but if you're not familiar with languages that answer these questions in different ways you might not be able to conceive of these misunderstandings.

Another you thing you can do to build PSL empathy is listen. Listen to questions and notice patterns in them. If you work with grad students who know R and are learning Python, try to notice what questions repeatedly come up.

In a general sense, this is what my favorite PyCon speaker [Ned Batchelder](https://nedbatchelder.com/) does a wonderful job of. Ned is a saint who spends thousands of hours in the #python irc channel repeatedly answering the same questions about Python. He does a bunch of other things like run the Boston Python Users Meetup group, and he coalesces all this interaction into talks which concisely hit all the things that are confusing about whatever that year's PyCon talk is.

The final idea for building Py2ndLang empathy I'll suggest is learning the language that your collaborator knows better so you can better imagine what their experience might be like. If you colleague is coming from Java, go learn Java! For this talk I did a mediocre job of learning C++ and Java. I did some research so I could try to present to you some of the things that could be tricky if you're coming to Python from one of these languages.
I chose these languages because they're common language for my colleagues. It's very reasonable to assume that a programming language will work like a language you already know, because so often they do! But then when there's a difference it's surprising.

C++ and Java are not my background! While Python was the first language I really got deep into, I had previous exposure to programming that colored my experience learning Python. My first programming language was TI-81 Basic, then some Excel that my mom taught me. In the Starcraft scenario editor you could write programs with a trigger language, so I did some of that. In middle school I got to use Microworlds Logo, which was pretty exciting. I did a little Visual Basic, got to college and did some MATLAB and some Mathematica, and then I took a CS course where they taught us Python.

My misconceptions about Python were so different than other students', some of whom had taken AP Computer Science with Java in high school. The languages I learned were all dynamically typed languages with function scope, so I didn't have the "where are my types?" reaction of someone coming from Java.

Java and C++ are good languages to focus on because they're often taught in schools, so when interviewing or working with someone right out of undergrad it can be useful to try to understand these languages.


Before we get to a list of tricky bits, there are some thinks I won't talk about because I don't call then "tricky." Not that they aren't hard, but they aren't pernicious, they're misunderstandings that will be bashed down pretty quickly instead of dangerously lingering on. New syntax like colons and whitespace, new keywords like yield; Python gives you feedback in the form of SyntaxErrors about the first group, and there's something to Google for with the second. When you first see a list comprehension in Python, you know there's something not quite normal about this syntax, so you know to research it or ask a question about it.

# Tricky Python From Java/C++ Cheatsheet

Let's split things that are tricky about Python for people coming from Java or C++ into three categories: things that look similar to Java or C++ but behave differently, things that behave subtly differently, and "invisible" things that leave no trace. The first category is tricky because you might not think to look up any differences, the second because you might test for differences and at a shallow level observe none when in fact some lurk deeper. The third is tricky because there's no piece of code in the file you're editing that might lead you to investigate. These are pretty arbitrary categories.

## Look similar, behave differently

### Decorators
There's a think in Java called an annotation that you can stick on a method or a class or some other things. It's a way of adding some metadata to a thing. And then maybe you could do some metaprogramming-ish stuff where you look at that metadata later and make decisions about what code to run based on them. But annotations are much less powerful than Python decorators.

{{< highlight python >}}
>>> @some_decorator
... def foo():
...     pass
... 
>>> foo
<quiz.FunctionQuestion object at 0x10ab14e48>
{{< / highlight >}}

Here (in Python) a python decorator is above a function, but what comes out is an instance of a custom class "FunctionQuestion" - it's important to remember that decorators are arbitrary code and they can do anything. Somebody coming from Java might miss this, thinking this is an annotation adding metadata that isn't transforming the function at definition time.

### Class body assignments create class variables
I've seen some interesting cool bugs before because of this. The two assignments below are two very different things:

{{< highlight python >}}
class AddressForm:
    questions = ['name', 'address']

    def __init__(self):
        self.language = 'en'
{{< / highlight >}}

`questions` is a class attribute, and `language` is an instance attribute. These are ideas that exist in Java and C++ with slightly different names (`questions` might be called a "static" variable, and `language` called a "member" variable), but if you see something like the top in one of those languages people might assume you're initializing attributes on an instance; they might think the first thing is another way of doing the second.

### Run-time errors, not compile-time
Here I've slightly misspelled the word "print:"

{{< highlight python >}}
if a == 2:
    priiiiiiiiiiiiint("not equal")
{{< / highlight >}}

This is valid Python code, and I won't notice anything unusual about it until a happens to be 2 when this code runs. I think people coming from languages like Java and C++ with more static checks will get bitten by this before too long and get scared of it, but there are a lot of cases for them to think about.

{{< highlight python >}}
try:
    foo()
except ValyooooooooooError:
    print('whoops)'
{{< / highlight >}}

Here's I've slightly misspelled `ValueError`, but I won't find out until foo() raises an exception.

{{< highlight python >}}
try:
    foo()
except ValueError:
    priiiiiiiiiiiiiiint('whoops)'
{{< / highlight >}}

Here `ValueError` is fine, but the code below it won't run until `foo()` raises an exception.

### Conditional and Run-Time Imports

Particularly scary examples of the above issue feature `import`s because people may think imports work like they do in Java or C++: something that happens before a program runs.

{{< highlight python >}}
try:
    foo()
except ValueError:
    import bar
    bar.whoops()
{{< / highlight >}}

It's not until `foo()` raises a `ValueError` that we'll find out whether the bar module is syntactically valid because we hadn't loaded it yet, or whether a file called bar.py exists at all!

### Block Scope

This might blow your mind if you're mostly familiar with Python: there's this idea called block scope. Imagine that every time you indented you got a new set of local variables, and each time you dedented those variables went away. People who use Java or C++ are really used to this idea, they really expect that when they go out of a scope (which they use curly brackets to denote, not indentation) that those variables will go away. As Python users, we might know that in the below,

{{< highlight python >}}
def foo():
    bunch = [1, 2, 3, 4]
    for apple in bunch:
       food = pick(apple)

    print(apple)
    print(food)
{{< / highlight >}}

the variables `apple` and `bunch` "escape" the for loop, because Python has function scope, not block scope! But this sneaks up on people a lot.

### Introducing Bindings

This above is sort of a special case of something Ned Batchelder has [a great talk about](https://nedbatchelder.com/text/names1.html), which is that all the statements below introduce a new local variable X:

{{< highlight python >}}
X = ...
for X in ...
[... for X in ...]
(... for X in ...)
{... for X in ...}
class X(...):
def X(...):
def fn(X): ... ; fn(12)
with ... as X:
except ... as X:
import X
from ... import X
import ... as X
from ... import ... as X
{{< / highlight >}}

(these examples taken from the talk linked above)

`import` in a function introduces a new local variable only accessible in that function! Importing in Python isn't just telling the compiler where to find some code, but rather to run some code, stick the result of running that code in a module object, and create a new local variable with a reference to this object.

## Subtle behavior differences

### Assignment
Python `=` is like Java, it's always a reference and never a copy (which it is by default in C++).

### Closures
A closure is a function that has references to outer scopes. ([mostly - read more](http://ballingt.com/python-closures/))
C++ and Java have things like this. Lambdas in C++ require their binding behavior to be specified very precisely, so each variable might be captured by value or by reference or something else. So a C++ programmer will at least know to ask the question in Python, "how is this variable being captured?" But in Java the default behavior is to make the captured variable final, which is a little scarier because a Java programmer might assume the same about Python closures.

### GC

It's different! We have both reference counting and garbage collection in Python. This makes it sort of like smart pointers in C++ and sort of like garbage collection in Java. And `__del__` finalizer doesn't do what you think it does in Python 2!

### Explicit super()

In Java and C++ there exist cases where the parent constructor for an object will get called for you, but in Python it's necessary to call the parent method implementation yourself with `super()` if a class overrides a parent class method. Super is a very cooperative sort of thing in Python; a class might have a bunch of superclasses in a tree and to run all of them requires a fancy method resolution order. This works only so long every class calls super.

I'll translate this one to the transcript later - for now you'll have to watch it because the visual is important: [explicit super](https://youtu.be/kWNBK2cnaYE?t=957).

## Invisible differences

### Properties and other descriptors

It can feel odd to folks coming from C++ or Java that we don't write methods for getters and setters in Python; we don't have to because ordinary attribute get and set syntax can cause arbitrary code to run.

{{< highlight python >}}
obj.attr
obj.attr = value
{{< / highlight >}}

This is in the invisible category because unless you go to the source code of the class it's easy to assume code like this only reads or writes a variable.

### Dynamic Attribute Lookup

Attribute lookup is super dynamic in Python! Especially when writing tests and mocking out behavior it's important to know (for instance) that a data descriptor on a parent class will shadow an instance variable with the same name.

### Monkeypatching

Swapping out implementations on a class or an instance is going to be new to people. It could happen completely on the other side of your program (or you test suite) but affect an object in your code.

### Metaprogramming

It takes less characters in Python!

{{< highlight python >}}
get_user_class("employee")("Tom", 1)
{{< / highlight >}}

The code above returns a class object based on the string "employee" and then creates an instance of it. It might be easy to miss this if you expect metaprogramming to take up more lines of code.

### Python 2 Whitespace Trivia

A tab is 8 spaces in Python 2 for the purposes of parsing significant whitespace, but is usually formatted as 4!

# What do we do with this?

Should we try to teach everyone all these things right now? Maybe not! If someone is interested, sure. But I think it's hard to hit all of these without much context. And be careful not to assume people don't know these things, maybe they do know 80% of them. I think this cheat sheet presents things that are important to be aware of while teaching whatever other topic is most pedagogically appropriate.

{{< youtube id="hY14Er6JX2s" >}}

I don't have time to talk much about teaching, so I'll point to [Sasha Laundy's talk](https://www.youtube.com/watch?v=hY14Er6JX2s) (embedded above) which I love, and quickly [quote Rose Ames](http://rose.github.io/posts/measured-in-wats/) and say that "knowledge is power; it's measured in wats." I think a great way to broach a misunderstanding is to present someone with a short code sample "wat" that demonstrates a misconception exists without necessarily explaining it because often all someone needed was to have the a flaw in their model pointed out to them.

Code review is a great impetus for sending someone such a wat. I don't have time to talk about code review so I'll point to [this terrific post by Sandya Sankarram about it](https://medium.freecodecamp.org/unlearning-toxic-behaviors-in-a-code-review-culture-b7c295452a3c).

Another thing we can do with this information is to write it in code comments. I think of comments as the place to explain why code does a thing, not to explain what that code is doing. But if you know what's happening in your code might surprise someone less familiar with Python, maybe you should say what it's doing? Or maybe you should write simpler code and not do that interesting Python-specific thing.

In the same way Python library authors sometimes write code that straddles Python 2 and 3 by behaving the same in each, imagine writing Python code that, if it were Java or C++, would do the same thing. Perhaps you'd have quite unidiomatic code, but perhaps it'd be quite clear.

![](/assets/pythongrowth.png)

image from this [Stack Overflow blog post](https://stackoverflow.blog/2017/09/06/incredible-growth-python/)

Python is becoming more popular. Maybe this means more people will understand it, and we'll get to use all our favorite Python-specific features all the time! Maybe this will mean Python becomes the lingua franca which ought to be as simple and clear as possible. I imagine it will depend on the codebase. I think as a code base grows tending toward code that is less surprising to people who do not know Python well probably makes more sense.

One final use for this cheat sheet is interviewing: interviewing is a high time pressure communication exercise where it really can help to try to anticipate another person's understanding of a thing. Candidates often interview with Python, but know C++ or Java better. If I can identify a misunderstanding like initializing instance variables in the class statement, I can quickly identify it, clarify with the candidate, and we can move on. Or perhaps I don't even need to if the context is clear enough. And when I'm interviewing at companies, it's helpful to remember what parts of my Python code I might need to explain to someone not as familiar with the language.
