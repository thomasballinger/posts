---
layout: post
title:  "Python and JavaScript"
date:   2014-3-17 10:00:00
---

One late night at Hacker School a few months ago, I had the chance to
discuss programming languages with the inimitable Will Byrd.
At one point I mentioned that JavaScript and Python felt terrifically similar to me.
Will thought that seemed odd, which made me want to examine it further.

Let me tell you about my favorite dynamically typed language.

JavaScript|Python
-----------------

In this language, everything is an object.[^object]

Objects are buckets of properties;[^1] they're mappings of names to
other objects[^same]. 
These properties can be accessed with dot notation - that is,
the name `bar` can be accessed on the object `foo` with `foo.bar`.[^namespaces]

Not all names that can be accessed with dot notation on an object are actually properties
of that object, but instead might belong to other linked objects.
Objects are linked in this way in a hierarchy for property lookup,
so if `bar` doesn't exist on `foo`, whatever object bar "inherits from" will be used
as another bucket of properties to look for that name in, and so on for that object.[^2] 
Most objects can have properties tacked on later, using `obj.prop = value`
syntax.[^tacked] Modifying or creating a new attribute in this way will only effect
the object to the left of the dot, even if using property access on the same
name would have retrieved a property from higher up the inheritance
tree.[^descriptors][^configurable]

Functions are a kind of object.
Functions have parameters which specify what names to bind passed arguments
to when executing the body of the function. 
Objects are returned from functions with the
`return` statement, and a default return value is used if no return statement
is encountered while executing the function.
There are no methods as such: no functions which are inherently bound to instances of objects.
Instead the method-ness of a function is determined at runtime.[^4]
When you look up a function on an object and
call it, its behavior is specific to that object.[^5]

Variables are references to objects, and are not constrained in the type of
object to which they can refer in any way. The only scope barriers are functions.[^barriers]
(not bodies of for loops, brackets, indentation, etc.)
A variable name may refer to a local variable or
a variable in a scope surrounding where the function was defined.

In some languages, while inside of a method definition, other methods of the same object
can be called without an explicit reference to an object.[^desirable]
If you want to refer to a method or instance variable from within a function,
you need to start with a reference to the object and do property lookup on that; 
there is no implicit object scope.

If a variable is declared in a function,
that name has been declared local for the whole function, even for uses of the name
occurring before the declaration of the variable.[^6]
If a variable from an outside scope can be accessed, it can be reassigned.[^7] 

Thanks for reading
------------------

What else can we say about both JavaScript and Python? Thanks to
several folks at [Hacker School](https://www.hackerschool.com/) for help.

[^1]: Python calls them attributes, and uses "property" to describe a specific
    kind of attribute.

[^2]: Python has classes, which means that non-class objects aren't allowed to be
    in the hierarchy of objects that attributes are looked up on. The
    attribute lookup process is complicated in Python ([this
    document](http://www.cafepy.com/article/python_attributes_and_methods/python_attributes_and_methods.html#attribute-search-summary) is a great resource), but it's basically to look for the attribute on the object,
    then the objects's class object, then the class objects that class
    inherits from.

    In JavaScript, the heirarchy is simpler: property lookup proceeds to an
    objects "prototype", and then on to that prototype's prototype, and so on.
    Prototypes are determined by the value of value of the `.prototype`
    property of the function that constructed the object via the `new`
    keyword.

[^object]: This is the biggest lie in the post. If you swallow it, you'll be
    alright with the other ones in the post.
    If this assertion offends you terrifically, you probably won't like this post.
    (or perhaps you'll enjoy nitpicking it :)
    If you can kind of see how that could be true, since most primitive types get
    automatically wrapped as their corresponding objects pretty seamlessly when
    you call methods on them, you might be able to keep reading.
    Please also ignore null and undefined are also objects, whose existance
    makes really derails this post.
    If this is new and intriguing, I found [this](http://javascriptweblog.wordpress.com/2010/09/27/the-secret-life-of-javascript-primitives/) a good intro to JavaScript primitives.
    What I'll continue to call objects in this posts are really objects plus
    JavaScript primitives.

[^4]: In Python, there are only functions, which can get curried into being
    bound methods at attribute lookup time if they are retrieved by
    property lookup on an object.
    In JavaScript, any function can be a method, if it is obtained *and
    immediately called* via property lookup. (`foo.bar()`, but not `a =
    foo.bar; a()`, which _does_ work in Python)

[^5]: In Python it's on lookup, in JavaScript it's on lookup-and-calling only
    if the two happen together. In Python the function is made specific to
    an object by partially applying the function with the first argument as
    the object - so the
    function takes one less argument, since the first positional argument is
    automatically pointed at the object. In JavaScript the keyword `this` now
    refers to the object.

[^6]: In JavaScript, variables are declared (their scoping is determined)
    with the `var` keyword. In Python, variables are declared by doing assignment
    (`a = 2`) or use of the `global` or `nonlocal` keywords.

[^7]: In Python, this means _not using the equals sign_ to declare the name as
    local, which how you reassign variables - so you're in a tricky situation.
    You can use `global` or `nonlocal` (Python 3 only) in a scope to specify that _even though
    you're using assignment_, that name refers to the variable with the same name
    in another scope.

[^namespaces]: In JavaScript, you can also use index notation, `foo['bar']`,
    to do the same thing. In Python, this notation is used to access an
    entirely separate namespace - this allows the keys of a dictionary
    to be entirely separate from its methods, so something like `foo.hasOwnProperty(bar)`
    isn't necessary when iterating over an objects properties.

[^descriptors]: This isn't true of [data descriptors](http://docs.python.org/2/howto/descriptor.html) in Python - in this case properties apparently later in the property lookup chain can steal a property lookup even when the earlier object has an attribute with that name - this is because [the real property lookup chain](http://www.cafepy.com/article/python_attributes_and_methods/python_attributes_and_methods.html#attribute-search-summary) is more complicated.

[^same]: or the same object

[^tacked]: In JavaScript, almost every object can have properties added later,
    but in Python some objects are backed by
    [`__slots__`](http://docs.python.org/2/reference/datamodel.html#slots)
    instead of dictionaries, so can't have arbitrary attributes added.

[^configurable]: Not true, because in JavaScript, setters and getters can be established for
    properties via [configurable properties](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)
    so getting or setting an attribute could run arbitrary code

[^desirable]: JavaScript is often written in a such a way that functions use
    local scope to provide access to variables specific to an object
    with with closures, but these variables aren't properties, they're just
    local variables

[^barriers]: And classes and modules in Python
