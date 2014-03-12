---
layout: post
title:  "Trees in Python"
date:   2014-03-09 11:12:17
---

This week I wrote a lot of trees:

* [Regular expression
  parser](https://github.com/thomasballinger/regexfun/blob/master/shorter.py) - builds regular expressions with a recursive
  descent parser 

* [Regression trees](https://github.com/thomasballinger/regressiontrees) - building a decision tree for classification

* Program Synthesis - I talked to a lot of other people about program
  synthesis and built things with some of them.

* [Simple Tree in Java](https://gist.github.com/thomasballinger/9453188) - helping some Java folks write code for some graph traversal
  exercises

I use trees a lot of course, but rarely write so many from scratch in a week.

For most of these, I found myself building up functions for a bit (as in the
top half of [this regression tree code](https://github.com/thomasballinger/regressiontrees/blob/master/test.py),
then starting to go object oriented once I got to the nesting part.

The pain point
that prompted this seemed to be writing recursive text representations of the
trees: I wanted `__repr__` magic so much that I'd give up my simple, transparent data
structures of tuples and dictionaries for opaque objects.

I'd also start with instance attributes when the tree was a single node, then
move to dynamically calculated properties once the property of a
thing starts to depend on its children trees.

Python does have some pain points for recursive data structures - the
inevitable `if empty` / `if self.left_child is None` constructions kept
popping up, taunting me that I wasn't using a language that actually
understood the different cases of my data structures, instead interpreting
every object as a bag of attributes I had to write custom code for.

Finally, tree transformations are nice! I wrote cleaner programs by building a
simple tree, then expanding or changing it, vs immediately building the full
tree.
