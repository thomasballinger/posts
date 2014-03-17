---
layout: post
title:  "Python and JavaScript"
date:   2014-3-17 10:00:00
---

This is still in a draft state - please don't link to it yet.

One late night at Hacker School a few months ago, I had the chance to
discuss programming languages with the inimitable Will Byrd.
As I described the languages with which I was most familiar,
I mentioned that Javascript and Python felt terrifically similar to me.
Will thought that seemed odd, which made me think want to examine it futher.
I've written this up as a more coherent description of how they're similar.

Let me tell you about my favorite dynamically typed language.

JavaScript|Python
---------------------

In this language, everything is an object.[^3]
Objects are buckets of properties;[^1] they're mappings of names to
objects. 
These properties can be accessed with dot notation - that is,
the name `bar` can be accessed on the object `foo` with `foo.bar`

Not all names that can be accessed with dot notation are actually properties
of that object, but instead might belong to other linked objects.
Objects are linked in this way by a heirarchy for property lookup,
so if `bar` doesn't exist on `foo`, whatever object bar "inherits from" will be used
as another bucket of properties to look for that name in, and so on for that object.[^2] 
Most objects can have properties tacked on later, using `obj.prop = value`
syntax. Modifying or creating a new attribute in this way will always effect
the object to the left of the dot, even if using property access on the same
name would have retrieved a property from higher up the inheritance tree.

Some objects are functions.
Functions have parameters which specifiy what names to bind passed arguments
to when executing the body of the function. Objects are returned from functions with the
`return` statement, and a default return value is used if no return statement
encountered while executing the function.
There are no methods as such (functions in which scope
rules work differently such that other properties of the object the method is
used on are automatically accessible), but instead the method-ness
of a function is determined at runtime.[^4] When you look up on an object and
and call it, sometimes something special happens to make the application of the
function behave differently, making its behavior specific to the object.[^5]

Variables are references to objects, and are not constrained in the type of
object to which they can refer in any way. The only scope barriers are functions.
(not bodies of for loops, brackets, indentation, etc.)[^scoping]
A variable name may refer to a local variable,
a variable in a scope surrounding where the function was defined, or a global
variable. There is no implicit object scope (see absence of "methods" above) -
if you want to refer to a method
or instance variable, you need to start with a reference to the object and to
property lookup on that. If a variable is declared somewhere in a function,
that name has been declared local for the whole function, even for uses of the name
occuring before the declaration of the variable.[^6]

Variables from scopes outside the current one can be reassigned if that
variables is declared to be from that scope.[^7]

To add:

* Hoisting!
* True vs true
* sytax: indentation vs brackets
* Built-in data structures vs nah, just dictionaries
* classical inheritance vs prototypal
* Two different namespaces for each objects, get item and get attributes
* regex in language vs library
* a ton more features / syntax
  * generators
* operator overloading
* undefined vs exceptions philosophy
* new vs class()
* see [note](http://www.cs.miami.edu/~burt/learning/five-easy-pieces/newwb/arrays_objects_dictionaries.html)

[^1]: Python calls them attributes, and uses "property" to describe a specific
    kind of attribute.

[^2]: Python has classes, which means that non-class objects aren't allowed to be
    in the hierarchy of objects that attributes are looked up on.

[^3]: Javascript has primitives?

[^4]: In Python, there are only functions, which can get curried into being
    bound methods at attribute lookup time if they are retrieved with dot notation
    property lookup on an object.
    In Javascript, any function can be a method, if it is obtained *and
    immediately called* via property lookup.

[^5]: In Python it's on lookup, in Javascript it's on lookup-and-calling when
    the two happen together. The something special in Python is that the
    function takes one less argument, since the first positional argument is
    automatically pointed at the object. In Javascript the keyword `this` now
    refers to the object.

[^6]: In Javascript, variables are declared (their scoping is determined)
    with the `var` keyword. In Python, variables are declared by doing assignment
    (`a = 2`) or the `global` or `nonlocal` keywords.

[^7]: In Python, this means _not using the equals sign_ to declare the name as
    local, which how you reassign variables - so you're in a tricky situation.
    You can use `global` or `nonlocal` in a scope to specify that _even though_
    you're using assignment, that name refers to the variable with that name
    in another scope.

