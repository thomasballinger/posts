---
date: 2016-06-28T11:12:17Z
tags: python
title: Basic banner grabbing with Python
url: /banner-grabbing-with-python/
---

I'm attempting to write a bot and I was trying to find some information
about ports on other computers. If anyone could help me out, I would appreciate it.

When you create a socket in Python, what does it represent?

What would be the underlying procedure for creating a connection with a host?

Let's say I try to connect to a host using socket.connect, does it raise an exception if it times out?

Does your typical at home router deny "unsolicited" connections?

How do I request the banners of the services running on a port if I do manage to connect?

---

You probably don't need to learn about sockets to make a bot, but you totally should anyway.
*I incorrectly assumed assumed we were talking about a Reddit bot which would automatically reply to
posts on that website. -Ed.*

> When you create a socket in Python, what does it represent?

A socket represents a connection to a computer.
So long as we're talking about the most common kind of socket,
it represents a unique set of your ip, their ip, the port you're using and the port they're using.

> What would be the underlying procedure for creating a connection with a host?

The server tells its operating system that it wants to accept connections on a
port: so not to ignore (block) those attempted connections.
The client program tells its operating system that it wants to connect to an ip address and a port.
The operating system makes it so with black magic.
(for more, read about TCP, but this would be ok to ignore until you've used them a bit)

> Let's say I try to connect to a host using socket.connect, does it raise an exception if it times out?

Yes, `socket.connect` will raise an error if it times out.

> How do I request the banners of the services running on a port if I do manage to connect?

I'm not sure what you mean by "banners." *I get to this later, but if you're
curious now a banner seems to be whatever a server sends you when you start a
TCP conversation with it on a port, or perhaps send the initial bytes of the
expected protocol. See [the Wikiedia article on Banner
grabbing](https://en.wikipedia.org/wiki/Banner_grabbing). -Ed.*


> Does your typical at home router deny "unsolicited" connections?

Outgoing connections (when a computer on your network opens a connection to a
computer not on your network, like `s = socket.socket(); s.connect(('google.com', 80))`
and incoming connections (when a computer not
on your network initiates a connection with one on your network) can be
considered separately. The former probably wouldn't be blocked, because

    s = socket.socket()
    s.connect(('google.com', 80))
    s.send(b'GET /\n\n')
    print(s.recv(10000))

is how your browser downloads web pages; if the router blocked the outgoing connect, this wouldn't work), but the latter could be.
It's hard to run a server locally that accepts incoming connections, and it
involves things like getting a static ip and port forwarding.
Most people cheaply rent machines that are always on and already set up with dedicated ip addresses somewhere far away instead (linode, ec2, etc.). Fortunately, for many things you don't have to; you can initiate the connections you need to make.

---

By banner I mean if you have a service running on the port, the banner would be the information regarding type of service, version, etc.

---

Do want this information on your own computer, or one you're talking to over the network?

If you want it on another computer, that's not something a raw TCP connection supports. All that sockets let you do is send bytes to another computer. You open a connection to a port on a remote computer and you start sending it bytes and you see what it sends back. There's no telling what it is - perhaps it's someone using [netcat](http://en.wikipedia.org/wiki/Netcat), a simple socket program that accepts one connection, prints whatever is sent, and lets you type whatever you want back. Perhaps it's [Apache](http://en.wikipedia.org/wiki/Apache_HTTP_Server) listening for HTTP requests from web browsers, and sending HTML documents, images and other files back. Perhaps it's FTP, etc.

There are standards:
[http://en.wikipedia.org/wiki/list_of_tcp_and_udp_port_numbers](http://en.wikipedia.org/wiki/list_of_tcp_and_udp_port_numbers)
shows which services are normally run on which ports.

---

So in the context of my question, procedurally, I need to connect to the host. Send some bytes? Then what would I do next in my program?

---

Disclaimer: if your goal is to quickly create a Reddit bot, you ought to follow a tutorial to do just that - something like http://praw.readthedocs.org/en/latest/index.html. You don't need to know that sockets even exist. It abstracts away sockets, abstracts away HTTP requests, and lets you build just the interesting parts.

But! If you'd like to play with sockets, then OK, and you're awesome.


You want to connect to "the host." I'm guessing this is www.reddit.com?

    s = socket.socket() #TCP is the default, so no arguments needed
    s.connect(('www.reddit.com', 80))  # From that wikipedia article before, web servers usually listen for connections on port 80

Great, now we have a connection. That's right, the next step is to send some bytes. The bytes we need to send are the bytes that make up an HTTP request, because we're going to talk to www.reddit.com using the HTTP protocol (see http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Example_session)

    s.send('GET / HTTP/1.1\nHost: www.reddit.com\n\n')

There! We've sent our bytes. These socket connections we have are two way, so we can now check to see if reddit.com has sent us anything back.

    print s.recv(1000)

    'HTTP/1.1 200 OK\r\nContent-Type: text/html; charset=UTF-8\r\nx-frame-options: S
    AMEORIGIN\r\nx-content-type-options: nosniff\r\nx-xss-protection: 1; mode=block\
    r\nServer: \'; DROP TABLE servertypes; --\r\nVary: accept-encoding\r\nDate: Fri,
     23 May 2014 05:05:41 GMT\r\nTransfer-Encoding:  chunked\r\nConnection: keep-ali
    ve\r\nConnection: Transfer-Encoding\r\n\r\n00006000\r\n<!doctype html><html xmln
    s="http://www.w3.org/1999/xhtml" lang="en" xml:lang="en"><head><title>reddit: th
    e front page of the internet</title><meta name="keywords" content=" reddit, redd
    it.com, vote, comment, submit " /><meta name="description" content="reddit: the
    front page of the internet" /><meta http-equiv="Content-Type" content="text/html
    ; charset=UTF-8" /><meta name="viewport" content="width=1024"><link rel=\'icon\'
    href="http://www.redditstatic.com/icon.png" sizes="256x256" type="image/png" /><
    link rel=\'shortcut icon\'href="http://www.redditstatic.com/favicon.ico" type="i
    mage/x-icon" /><link rel=\'apple-touch-icon-precomposed\'href="http://'

Great! We got something back. If we keep receiving more bytes, we'll find more and more.

    data = ''
    while True:
        d = s.recv(1000)
        if not d:
            break
        data += d

Now we've got a bunch of bytes back from reddit.com. They look kind of like
what you see if you right click the front page of reddit and click "View Page
Source," but there's some extra information at the top. If only we could
understand what they meant... *I imagined we might continue to talk about
HTTP, but we didn't. -Ed.*

----

Honestly, I posted in /r/learnpython asking questions about a port scanner I was building and only received a single downvote and nothing else for like 5-6 hours, so I thought people didn't like these sort of questions. I'm not building a bot unfortunately, but rather trying to learn about networking and programming on a low level.

[http://www.reddit.com/r/learnpython/comments/265vm0/port_scanner_for_learning/](http://www.reddit.com/r/learnpython/comments/265vm0/port_scanner_for_learning/)

Can I ask one more? I asked some time ago and people didn't like my question so much...

*I've cut this question about about why clicking on links is a security risk. -Ed.*

Also, don't feel your time is wasted here. Whenever I ask questions like this, I usually do a write up of the solution afterwards and post it on my website. I want the question to be available in future to people with similar interests for when they search. Also, it helps me learn and research more, so thank you.

----

Sorry, I assumed you were trying to build a Reddit bot because of your
username *[which was GoodBehaviorBot -Ed.]* and because that's what I think of when I hear the word "bot" on Reddit.

If you're building a port scanner, I'd start trying to connect to a server
(say www.reddit.com) on lots of different ports, and see on which ones you
could connect. If you can connect to one, you should "greet" it in a bunch of
protocols. If it responds to "GET /\n\n" an HTTP response (web page plus
headers) then it's something that speaks HTTP. If it says `220
smtp.example.com ESMTP Postfix` to you without you having to send anything to
it, then it's an SMTP server. If you receive bytes like `NOTICE AUTH :***
Looking up your hostname...` then you've found an irc server, etc. Definitely
take a shot at building this! The first iteration doesn't have to be more complicated that a loop over ip addresses and another inner loop over ports to find a list of ports that are accepting TCP connections.

*The conversation goes on for a bit about the security risks of clicking on a link, see the whole thing from this point at
[https://www.reddit.com/r/Python/comments/266aav/sockets/chp1zkn](https://www.reddit.com/r/Python/comments/266aav/sockets/chp1zkn).  -Ed.*

----

This covers similar information as [Talking to Other Computer with
Python](https://github.com/thomasballinger/talkingtoothercomputers/blob/master/talkingtoothercomputers.ipynb)
but follows a different format. It answers questions in the form they were
asked by an [anonymous internet poster](https://www.reddit.com/r/Python/comments/266aav/sockets/).

I think [Matt Might](http://matt.might.net/) and [Philip
Guo](http://www.pgbovine.net/) have both told me to write up answers to
questions you've gotten more than once. This is sort of like that, but
instead of trying to imagine an audience I've taken an actual (internet)
conversation I had with someone and posted it here.

I've been thinking about
how lectures and textbooks are difficult to write because they will
be received by diverse audiences, or rather how it's difficult for
a lecture or book to be as helpful to a learner as a dialog.
Although our communication was lacking in this conversation,
(I didn't understand the poster's purpose until very late in the
conversation), we were able to connect new knowledge to their
prior experience I wouldn't have been able to imagine were I concocting
a lesson.

I'm most interested in this post being [helpful to its audience, not in being right](https://twitter.com/b0rk/status/743390898810159104).
However I'd appreciate corrections or feedback on either front.
