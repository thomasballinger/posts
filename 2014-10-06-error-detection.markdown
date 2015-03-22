---
layout: post
title:  "Reading Error Detecting and Error Correcting Codes"
date:   2014-10-06 12:00:00
alias:  /2014/10/06/error-detection.html
---

Last week's [Hacker School](https://www.hackerschool.com)
[Paper of the Week](https://www.hackerschool.com/blog/44-paper-of-the-week-error-detecting-and-error-correcting-codes)
was Richard Hamming's ["Error Detecting and Error Correcting
Codes."](http://www.lee.eng.uerj.br/~gil/redesII/hamming.pdf)
I worked through the paper (not something I do a lot)
and wanted to show an example of my process reading it.

[I'm probably an "active
learner,"](http://blog.melchua.com/2014/08/12/learning-styles-for-programmers-activereflective/) insofar as that's a useful label in describing learning styles, [whatever those are](http://blog.melchua.com/2013/06/19/hacker-school-session-engineering-learning-styles/).[^learning-styles]
The way I worked through the paper was by reading a bit for a problem description,
trying to solve the problem myself, then learning why my solution was suboptimal
or reading the solution to a problem I couldn't solve.
This is like attempting the exercises in a math text before reading the chapter;
it helps me care about solutions to problems because I want to compare my solution,
or I've proved I can't solve them myself.

Here's some of that process for the first two sections of the paper.
It's humbling to share rough ideas and even rougher programs that were knocked
down pretty hard only a few minutes after I wrote them out.

# Efficient Error Detection
We could obviously detect errors if we sent all the information twice - if the
two messages aren't the same, we've got a problem.
But we should be able to be more efficient about this with a checksum, and we
can mod it to make it a parity bit.

## After reading

Yep! Looks like we're on the same page.

# Single Error Correcting Codes

The most naive way I can think of for detecting errors is to send the
information three times - if a bit is the same in two of the copies but not
the third, that's an error. This method might have some nice properties about
the errors being likely to be independent from each other, but in the problem described
we're dealing only with recovering from a single error, and surely we can do
better.
Two copies of the information, each with a checksum would be better. We don't
really need the second checksum if we're assuming just one error.
But a series of parity checks should be enough as well - and we might not need
as many.

My first try with parity bits was 4 bits, with two different parity checks
applying to each bit of data:

    def parity(bits):
        return sum(bits) % 2

    def secc4(info):
       checks = [parity(bits) for bits in [info[0:2], info[2:4], info[0:4:2], info[1:4:2]]]
       return info + checks + parity(info)

    secc4([1, 0, 0, 1])
    #[1, 0, 0, 1, 1, 1, 1, 1]

This still took as much space as repeating the information, but the
idea of combining information from several parity checks
to locate the bad bit seems good. Next I could do a tictacttoe-like
arrangement:

    def secc9(info):
        assert len(info) == 9
        checks = ([parity(info[i:9:3]) for i in range(3)] +
                  [parity(info[i:i+3]) for i in range(0, 8, 3)])
        return info + checks + parity(info)

    secc9([1, 0, 0, 1, 0, 0, 1, 1, 1])
    #[1, 0, 0, 1, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]

Now our check bits grow more slowly than the info. The general case:

    import math

    def secc_m(n):
        #was stuck on the math for a while, so doing this the slow, straightforward way
        if n == 0: return 0
        for i in xrange(int(math.ceil(math.sqrt(n))) + 1):
            k = 2 * i + 1
            m = n - k
            if m < 0:
                raise ValueError("bad n")
            if i == math.ceil(math.sqrt(m)):
                return m

    def secc(info):
        num_rows = math.ceil(math.sqrt(len(info)))
        checks = ([parity(info[i:len(info):num_rows]) for i in range(num_rows)] +
                  [parity(info[i:i+num_rows]) for i in range(0, len(info), num_rows)])
        return info + checks + parity(info)

    def decode_secc(data):
        """
        >>> decode_secc('001100110')
        [0, 0, 1, 1]
        >>> decode_secc('001100111')
        Traceback (most recent call last):
        ValueError: bad parity check, but nothing to fix
        >>> decode_secc('001000110')
        [0, 0, 1, 1]
        >>> decode_secc('000100110')
        [0, 0, 1, 1]
        >>> decode_secc('011100110')
        [0, 0, 1, 1]
        >>> decode_secc('101100110')
        [0, 0, 1, 1]
        >>> decode_secc('111100110')
        [1, 1, 1, 1]
        """
        if isinstance(data, str):
            data = [int(x) for x in data]
        m = secc_m(len(data))
        info = data[:m]
        checks = data[m:]
        par = data[-1]
        if parity(info) == par:
            return info
        k = len(data) - m
        num_rows = (k - 1) / 2
        rows = [parity(info[i*num_rows:(i+1)*num_rows]) for i in range(num_rows)]
        columns = [parity(info[i:m:num_rows]) for i in range(num_rows)]
        row_check = [exp == calc for exp, calc in zip(data[m:m+num_rows], rows)]
        col_check = [exp == calc for exp, calc in zip(data[m+num_rows:m+2*num_rows], columns)]
        if len([c for c in row_check if not c]) > 1 or len([c for c in col_check if not c]) > 1:
            raise ValueError('more than one row or col bad: %r %r' % (row_check, col_check))
        if len([c for c in row_check if not c]) != len([c for c in col_check if not c]):
            raise ValueError('only one of (row, col) bad: %r %r' % (row_check, col_check))
        for i, (r, c) in enumerate((r, c) for r in row_check for c in col_check):
            if not r and not c:
                info[i] = (info[i] + 1) % 2
                if parity(info) == par:
                    return info
                else:
                    raise ValueError('parity check fails after fixing data: %r' % (info,))
        else:
            raise ValueError('bad parity check, but nothing to fix')

This tictactoe method needs `2*ceil(sqrt(num_bits))` bits for correcting checks
and another bit for parity on either the orig data or the checks. I had
trouble solving m given n because math, so iterate through the possible
values to check them.

##After reading

Increasing the dimensionality was a good thought, but I should have played
with this more - obviously increasing the dimensionality is better than making
the tictactoe board bigger. I wish I'd made the jump to the 3D or 4D tictactoe board,
and then might have seen that increasing the dimensionality is more efficient that using more values per dimension, and
might have made the jump to using only two values per and the natural
correspondence this has to binary bits.

Naturally I absolutely loved the unit hypercube explanation later in the paper.

The starting with information theory technique (having enough bits to store the
information we know we need to store) that the paper uses
is a cooler way to go about this - I wanted a concrete method for storing the
information first, but establishing a bound like this would have been smart,
and it made everything fall into place so beautifully.

[^learning-styles]: I'm sympathetic to criticism of models like this,
    but happen to find this one useful for remembering and categorizing learners I
    work with, which is something I [spent a lot of time
    doing](https://www.hackerschool.com/about) and am really thankful to
    [Mel](http://melchua.com/) for introducing me to it.
