---
aliases:
    - /2014/10/14/yield-from.html
    - /2014/10/14/yield-from
date: 2014-10-14T11:00:00Z
tags: python
title: In celebration of yield from
url: /yield-from/
---

A conversation with current Hacker Schooler [Cerek](https://github.com/crockeo)
about writing a
boggle solver in Haskell motivated me to squeeze in writing up a
solution in Python yesterday morning between appointments with
Hacker Schoolers.

The strategy I followed was to generate all candidate
words and check them for validity with a dictionary, with the optimization
of not actually traversing the entire tree of possible words.
I wanted to separate building the candidate words and
checking them for validity, but allow the two processes to communicate to
eliminate branches that weren't
promising: if no English words begin with `abcd` don't bother investigating
the branch of the tree of candidate words that begin with these letters, like
`abcdh`. Coroutines seemed like a good fit.

I started coding in Python 2 because that's still my default for throw-away
scripts, but ended up with code like this in a few places:

    def candidates(board, row, col, visited, candidate=''):
        if (yield candidate): # whether this branch is a dead end
            return
        for c, (y, x) in unvisited_neighbors(board, visited, row, col):
            visited.add((y, x))
            cands = candidates(board, y, x, visited, candidate+c)
            for cand in cands: break
            while True:
                try:
                    cand = cands.send((yield cand))
                except StopIteration:
                    break
            visited.remove((y, x))

Then remembered - of course, `yield from`! I should be using Python 3 for
this:

    def candidates(board, row, col, visited, candidate=''):
        if (yield candidate): # whether this branch is a dead end
            return
        for c, (y, x) in unvisited_neighbors(board, visited, row, col):
            visited.add((y, x))
            yield from candidates(board, y, x, visited, candidate+c)
            visited.remove((y, x))

Having independently come up with something pretty close to the functionality
of `yield from` solidifies how that construct works for me.
See [the vim `:TOhtml`-generated diff](/assets/yield-from-diff.html) for the full code.[^1]
One more reason to be using Python 3 by default! :)

[^1]: The ugly timing code was for figuring out that the Python 3 code was 40% faster.
      It'd be fun to investigate where this came from. Yes, I had to [double check](http://math.stackexchange.com/questions/27202/what-does-x-faster-mean)
      the meaning of "40% faster" before using it here, and don't think it's a
      very clear thing to say.
