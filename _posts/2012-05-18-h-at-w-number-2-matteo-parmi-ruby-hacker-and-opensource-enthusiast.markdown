---
layout: post
title: "H@W #2 - Matteo Parmi: ruby hacker and opensource enthusiast"
date: 2012-05-21 17:27
comments: true
published: true
featured: false
tags: builders hackers interview h@w
hn: 
rd: 
---

Hi guys, the second [Hackers @ Work](http://armoredcode.com/blog/new-monothematic-posts-serie-hackers-at-work/)
interview is with [Matteo Parmi](http://parmi.it).

Matteo is a friend of mine and we worked together in
[Bitmama](http://www.bitmama.it) a web agency here in Milan, Italy.

He was the developer team leader, a perfect CTO for any startup if we were in
the Valley. He is a great developer and he loves discovering the web for new
technologies and new programming patterns to apply in his projects.

<!-- more -->

## The Interview
<span class="question">
"Hi Matteo, welcome to Hackers @ Work interviews series, can you please
introduce you to our readers telling us about your background and who are you?"
</span>
<span class="answer">
I'm a lazy boy trying to get some money doing what I love to do. I'm a web
application developer since 12 years ago. I get across all the Internet ages
but these days I'm love with ruby and all interpreted programming languages.
</span>

<span class="question">
"As a developer, can you describe your setup in terms of hardware
and sure also software, editors, IDE, energy drinks, music you listen
when you code, … ?"
</span>
<span class="answer">
I'm a Mac Os X user but I'm thinking about switching back to Linux operating
system. The iTerm is almost the only application running fullscreen, with vim
for source code writing and Google Chrome Javascript console.
I'm mainly a Rock/Metal fan and that's my music background while coding.
However, I'm discovering more relaxing harmonies to get me more productive
while coding. 
</span>
<span class="question">
"Which are the key factor for a successful web application?"
</span>
<span class="answer">
Good question. If we talk about code quality, the web application must be
development with evolution in mind. Developers must be free to make changes, to
refactor existing code without woo much effort.

It definitely must have a complete test bed and testing methods must be easy to
write and to apply. Mainly to involve developers in writing tests.

Quick tests make software refactoring an easy do adopt practice.

As far it concern the successful in terms of users... well I'd like to know it
myself as well =).
</span>
<span class="question">
"What's your toughts about introducing security in the software development
lifecycle? Do you think it will help? Which can be the tradeoffs to reach?
</span>
<span class="answer">
Security is crucial in web applications. I can't imagine a template language
that doesn't care about output canonicalization and sanitize. 
On the other side, an ORM must take care about SQL Injections attacks and web
framework to be resilient to form forgery and parameter tampering.

It's all about frameworks in my opinion. Everyday, every software intended for
developers should take care about security.

As you know, it's so simple to introduce security bugs in you code. As example,
think about the github and rails based applications security vulnerabilitity so
famous a month ago.

That was a security bug introduced by developers not making defensive
programming instead of a framework defect.
</span>
<span class="question">
Opensource: let's talk about your work as contributor / enthusiast. Which
projects do you find more interesting or which projects do you contribute most?
</span>
<span class="answer">
I release most of all my work as [opensource](https://github.com/tejo). More
complex projects are private and I'm afraid of that. 
However I try to refactor all of those private projects to bring some general
purpose libraries out of the box and so I can open that code on
[github](https://github.com/tejo).

I think that reading other people source code is a great way to improve your
skills as developer.

The great _revolution_ in the last 5 years (from the opensource ecosystem point
of view) I think it has been [github](http://github.com).
I yet remember the dark ages about [sourceforge](http://www.sourceforge.net)
and svn... brrrrr!
</span>
<span class="question">
The Web: do you see any interesting technology to go for the stage? Which ones
will be the winning framework/project combos for 2012?
</span>
<span class="answer">
I think [node.js](http://node.js) it must be a great pretender for 2012 Web
scene.
Having the chance to write both client than server side code one time is an
appealing idea.

We have to admit that it's even fast, a test bed with 50 cases with node.js it
has been executed in a couple of seconds. Impressive.

You don't even need a strategy to reload a source file after modifications, you
just restart the server (50ms boot time) and you're ready to go. Actually I'll
think twice before using node.js in a complex and strategic project. With Ruby
and Rails you've get all what you need ready out of the box.

However the community around node.js is very active and both tools than
libraries are written and released with an impressive speed.

[Coffeescript](http://coffeescript.org/) is a surprising technology as well. I use it in all my projects
nowadays, I can't remember I wrote a line of vanilla javascript latest year and
I even don't know how to start it again =).
</span>
<span class="question">
Node.js, socket.io, html5... technologies are moving towards the client.

Are we moving back to an seventies client-server approach with a common solid
protocol like http?
</span>
<span class="answer">
A lot of work is yet to be done in this direction... http is not that efficient
or at least it's not so responsive to act as backend protocol for realtime
client-server applications.

There are a couple for good written alternatives that are really promising like
[SPDY](http://dev.chromium.org/spdy/spdy-whitepaper) and [Websockets](http://www.websocket.org/) but, answering to the question, I think we have to wait
some time to reach a seventies fashioned client-server model.
</span>
<span class="question">
Matteo, help a security specialist to help you. How can we improve your code
with security bits, giving you hints with a gentle approach without slowing you
down?

What do developers really need?
</span>
<span class="answer">
We need tools that calculates metrics, and that they perform code review and
static analysis. Ruby scenario is moving quickly towards helping developers in
improving quality (with tools to discover duplicated code), but as far as today
I can see nothing about security analysis for the source code.

A reason can be that a dynamic language is not so easy to being statically analyzed.
</span>
<span class="question">
That's all Matteo, thank you so much for your patience and time.
Before we leave, feel you free to close with an opinion, a thoughts,
whatever argument you want to close with. You have a tweet (140
chars)… from now:
</span>
<span class="answer">
_stay satisfied stay wise_
</span>

## End of file

Thanks [Matteo](http://parmi.it) for being hosted in this Monday edition for
Hackers @ Work. We are 3 days late for some personal time issues, my fault
sorry.

Matteo pointed out some very interesting points of view. 

Web is pushing towards the code for being executed client side in order to
overcome HTTP protocol inefficiencies.

Security must drive this change covering client side technologies and new way
to write web applications. Even classical web penetration testing techniques
must evolve in order to make sure we can find vulnerabilities in a node.js
powered web application or in another client-side framework written app.

It seems testing to be fast, automated and reliable is crucial ina a developer
workout. 
Security specialists need to move over Powerpoint or attack tools in order to
fill test cases with security checks and even help people writing frameworks to
embed security into them.

Rails make a great work in to this direction introducing a couple of years ago
anti cross site request forgery token. 
This is the lead direction to follow.

Enjoy it.
