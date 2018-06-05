---
date: 2017-04-04T18:30:00Z
title: Debouncing terminology
url: /debouncing/
---

<div class="asking-questions-intro"></div>

When I get stuck on a computer thing, I tend to ask a question
in an irc channel or Slack or Zulip.

<style>
  .post div {
    margin: 0 auto;
  }
  .asking-questions-intro {
    width: 510px;
    height: 400px;
    background: url("/assets/drawings/7c4f1317-631d-40b4-8ccd-249653574024.png") no-repeat;
    background-position: left -170px top -130px;
  }
  .asking-questions-simple {
    width: 250px;
    height: 120px;
    background: url("/assets/drawings/98dea813-3f23-4b06-88b7-241cde1b9926.png") no-repeat;
    background-position: left -50px top -30px;
  }
  .asking-questions-multiple {
    width: 300px;
    height: 240px;
    background: url("/assets/drawings/80addbcf-f5d9-4e5b-ba3d-f9bca5ec43f0.png") no-repeat;
    background-position: left 0px top -30px;
  }
  .asking-questions-cancel-nevermind {
    width: 300px;
    height: 100px;
    background: url("/assets/drawings/98dea813-3f23-4b06-88b7-241cde1b9926.png") no-repeat;
    background-position: left -30px top -210px;
  }
  .asking-questions-cancel-ignore {
    width: 300px;
    height: 150px;
    background: url("/assets/drawings/98dea813-3f23-4b06-88b7-241cde1b9926.png") no-repeat;
    background-position: left -30px top -320px;
  }
  .asking-questions-cancel-still-use {
    width: 300px;
    height: 170px;
    background: url("/assets/drawings/98dea813-3f23-4b06-88b7-241cde1b9926.png") no-repeat;
    background-position: left -30px top -490px;
  }
  .asking-questions-queued-up {
    width: 400px;
    height: 350px;
    background: url("/assets/drawings/98dea813-3f23-4b06-88b7-241cde1b9926.png") no-repeat;
    background-position: left -500px top -80px;
  }
  .asking-questions-queued-up-cancel {
    width: 400px;
    height: 240px;
    background: url("/assets/drawings/98dea813-3f23-4b06-88b7-241cde1b9926.png") no-repeat;
    background-position: left -350px top -860px;
  }
  .asking-questions-concurrent {
    width: 320px;
    height: 250px;
    background: url("/assets/drawings/98dea813-3f23-4b06-88b7-241cde1b9926.png") no-repeat;
    background-position: left -480px top -480px;
  }
  .asking-questions-throttling {
    width: 380px;
    height: 270px;
    background: url("/assets/drawings/80addbcf-f5d9-4e5b-ba3d-f9bca5ec43f0.png") no-repeat;
    background-position: left -580px top -10px;
  }
  .asking-questions-debouncing {
    width: 200px;
    height: 290px;
    background: url("/assets/drawings/80addbcf-f5d9-4e5b-ba3d-f9bca5ec43f0.png") no-repeat;
    background-position: left -10px top -520px;
  }
  .asking-questions-debouncing-after-first {
    width: 350px;
    height: 500px;
    background: url("/assets/drawings/80addbcf-f5d9-4e5b-ba3d-f9bca5ec43f0.png") no-repeat;
    background-position: left -340px top -500px;
  }
  .asking-questions-queue {
    width: 250px;
    height: 470px;
    background: url("/assets/drawings/80addbcf-f5d9-4e5b-ba3d-f9bca5ec43f0.png") no-repeat;
    background-position: left -320px top -10px;
  }
</style>

<div class="asking-questions-simple"></div>

Sometimes it takes a while for my question to be answered. This is OK!
Sometimes the act of asking the question helps me get unstuck.
Often I end up making some progress after asking the question.
At this point I can say "nevermind, I don't need this question answered
anymore,"
<div class="asking-questions-cancel-nevermind"></div>
or I can wait for an answer anyway.
Sometimes even if I say I don't need an answer,
someone will have been typing up a response already and
be excited to share it,
so I can ignore the their response instead.
<div class="asking-questions-cancel-ignore"></div>
And sometimes the answer is still useful - maybe they suggest a different
solution than the one I found!
<div class="asking-questions-cancel-still-use"></div>

---

Often before I get the answer back, I make enough progress to come up with a new question!

<div class="asking-questions-queued-up"></div>

If I'm asking a specific person these questions, it might especially make sense to
cancel the first question so they can focus on my next question.

<div class="asking-questions-queued-up-cancel"></div>

But if it's to a group, this might not get me my second answer any quicker.
I might as well read these answers to the questions I've figured out.

<div class="asking-questions-concurrent"></div>

---

If I'm trying to value the question answerers' time, I might limit myself to
one question every 10 minutes. If I have more than one question while in my
10 minutes cool-down period, I'll ask the most recent one since it's the most
relevant.

<div class="asking-questions-throttling"></div>

If I want a response faster from a single question-answerer that doesn't listen
to me when I try to cancel previous questions,
I might wait a bit each time I have question to see
if I come up with another more relevant one.

<div class="asking-questions-debouncing"></div>

For a single dedicated question-answerer that won't stop what they're doing
to answer a new question, this might get me the answer to my
latest question more quickly on average, despite this added self-imposed
delay.

If getting a quick answer to an earlier question would still be somewhat useful, I might
immediately ask the first question, resorting to the waiting-before-asking strategy only
when the question-answer is busy answering a question.

<div class="asking-questions-debouncing-after-first"></div>

A similar approach ignores the time between questions completely, instead
keeping at most a single question in flight:

<div class="asking-questions-queue"></div>

If the transmission of questions and answers takes a significant amount of
time and I want to keep the question answerer as busy as possible to wring
the maximum amount of knowledge out of them, I might instead try to keep
up to two questions in flight; depending on the relative value of old
responses, the profile of question answering times, and the transmission times
of questions and answers.

---

I'm not actually talking about asking questions in this post,
although that's a great topic! [Julia] has written some [great
things](https://jvns.ca/blog/2016/08/31/asking-questions/)
[about that](https://jvns.ca/blog/good-questions/).

Instead this is about debouncing techniques!
I usually think of using them in JavaScript
to limit requests to a server, but they're more general than that.

I do have a real question though: what are these techniques called?

* cancelling a request: cancelling
* limiting to 1 request per time period: throttling
* waiting for fixed time after each request to replace: debouncing
* maintaing no more than 1 request in flight: ?
* debouncing, but no delay if no request currently in flight (letting the
  first request through un-debounced): ? ("debouncing, letting first request
  through," "debouncing while a request is out")
* maintaining no more than n requests in flight: ?

Let me know!

<a href="https://twitter.com/ballingt" class="twitter-follow-button" data-show-count="false">Follow @ballingt</a><script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>
