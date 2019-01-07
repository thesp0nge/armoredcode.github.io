---
layout: post
title: "H@W #1 - Simon Bennetts: Owasp Zap Project leader"
date: 2012-05-04 07:45
comments: true
published: true
featured: false
tags: breakers builders hackers interview h@w
hn: http://news.ycombinator.com/item?id=3927388
rd: http://redd.it/t6ii8
---

## The perfect mixin: a developer becoming an appsec specialist

When I planned the [Hackers @ Work](http://armoredcode.com/blog/new-monothematic-posts-serie-hackers-at-work/) 
serie post, I had no doubt for the post #1. 

Simon Bennetts is growing up in the [Owasp](http://www.owasp.org) community as
great contributor and outstanding project leader. Its 
[Owasp Zap](http://code.google.com/p/zaproxy) proxy is becoming a superb
opensource alternative to commercial application proxy (like paros, or burp) to
be used in real world web application penetration tests.  

<!-- more -->
## The interview

<span class="question">
"Hi Simon, and welcome to the first Hackers @ Work post for armoredcode.com.
Can you please introduce you to our readers telling us about your background
and who are you?"
</span>

<span class="answer">
Hi, and thank you for talking to me :)

I'm the project lead for the [OWASP Zed Attack Proxy](https://www.owasp.org/index.php/ZAP).
My background is in development, but in the last few years I've been
working more and more on application security.
So as well as developing security critical web applications I also
perform pentests and run security talks and courses.
</span>

<span class="question">
"As a developer, can you describe your setup in terms of hardware
and sure also software, editors, IDE, energy drinks, music you listen
when you code, ... ?"
</span>

<span class="answer">
I typically dont use a high spec machine - any modern machine should
be able to cope with the demands of developing software.
At the moment I have a Windows 7 box on my desk, which I use for email
and documents (Word/Excel/Powerpoint) and a Linux box (Fedora 16)
which I use for developing and pentesting.
I use Eclipse for most development, but every so often when I want to
do complex find and replace commands I go back to using vi!
I usually just drink tea at work, and for some reason I've never
managed to get the hang of working with music in the background.
</span>

<span class="question">
"As a security specialists, which are the most common pitfalls you
see in web apps when doing assessments?"
</span>

<span class="answer">
It actually varies a lot, depending on the security experience of the dev team.
I see some apps where security really hasn't been thought about, and
those are always very easy to break. 

I usually find most of the OWASP top ten in those sort of apps.

But some teams are much more security aware, and its then usually
application logic vulnerabilities that are the main problems.
</span>

<span class="question">
"What's you opinion about security in the SDLC? Something can be
improved or not? How?"
</span>

<span class="answer">
To be honest I think its a question of business priorities.

If businesses really want secure applications then they need to ensure
their staff have good security training, are given enough time to
consider security in their applications and perform security testing
as early as possible.
Just getting an external pentest one week before an application goes
live is not the way to go.
Most of the developers I talk to know they dont know enough about
security but are keen to learn more.
If they are given the opportunity to learn and understand that
security is a real priority then I think we'll start to see more
secure applications.
</span>

> I find I learn much more quickly if I get my hands dirty!

<span class="question">
"Owasp ZAP. When did you start it and which is the idea behind the
project? Is is suitable for developers?"
</span>

<span class="answer">
I launched it in September 2010, but I actually started working on it
about a year before that.

At that stage I wasnt thinking about releasing it, I just started
looking at the Paros source code (of which ZAP is a fork) to help me
learn more about security tools - I find I learn much more quickly if
I get my hands dirty!

I then started tweaking the code to fix issues and implement small
features that I wanted to use.

At the same time I was starting to give security training courses -
teaching developers basic pentesting techniques.
And the first question developers asked was "which tools should we use".
I had a good look around for ideal tool to recommend, but I couldn't
find anything which fitted the criteria I thought were important for
developers.

I realised my hacked version of Paros could become that tool, tidied
it up, and released it as ZAP.

So I specifically released it for developers and functional testers,
and was quite surprised when it was taken up so enthusiastically by
professional pentesters.
</span>

> If businesses really want secure applications then they need to ensure
> their staff have good security training, are given enough time to
> consider security in their applications and perform security testing
> as early as possible.

<span class="question">
"Help developers to understand how to use Owasp ZAP during their
tests."
</span>

<span class="answer">
Developers can use ZAP in 2 ways:

1. You can use it to create security regression tests - this is
  especially useful if you have existing tests that drive a web browser
  using something like Selenium.
  You need to proxy your regression tests via ZAP, then kick off the ZAP
  Spider and then the Active Scanner using the REST API.
  This will find common security vulnerabilities, and can be done as
  part of your continuous integration process.
  <br />
  I've documented an example of how this can work [here](http://code.google.com/p/bodgeit/wiki/RegTests)
  <br />
  We use ZAP in this way on security critical applications where I currently work.

2. You can also use it as a pentester does - proxying your browser via
  ZAP, exploring your application and then trying to find more
  vulnerabilities manually.
  <br />
  The [OWASP Testing Guide](https://www.owasp.org/index.php/OWASP_Testing_Guide_v3_Table_of_Contents) 
  is a good place to start if you want to go down this route.
</span>

<span class="question">
"How much time do you spend in hacking ZAP's code? There is strong
community behind? How many people does help you?"
</span>

<span class="answer">
At the moment I develop ZAP in my spare time, and theres never enough of that!
But I'm going to be joining the Mozilla Security team soon, so I'll be
able to work on ZAP and other cool open source security tools as part
of my job :D

There are [lots of people](http://code.google.com/p/zaproxy/wiki/HelpCredits)
involved in ZAP: but the main coding is done by a handful of developers.

And there are now 3 ZAP related Google Summer of Code projects, which
I'm really excited about.
</span>

<span class="question">
"Filling the gap between developers and application security,
purpose of this blog. Help me, tell me what do you think it must be
done to bring those worlds closer."
</span>

<span class="answer">
Developers need security training, and the time to make their projects secure.
Security must be considered as early as possible, and the sooner
security regression tests are introduced into a project the better.
And security people need to get involved much earlier on in projects.
They need to talk to developers, understand the pressures they are
under and convince senior managers that security needs much more time
and effort spent on it.
</span>

<span class="question">
"That's all Simon, thank you so much for your patience and time.
Before we leave, feel you free to close with an opinion, a thoughts,
whatever argument you want to close with. You have a tweet (140
chars)... from now:"
</span>

<span class="answer">
Appsec is still at a very early stage and has a long way to go. Anyone
who wants to can still make a big impact - what are you waiting for?


Many thanks!

Simon
</span>

## End of File

I'd like to thanks Simon for being the first [Hackers @ Work](http://armoredcode.com/blog/new-monothematic-posts-serie-hackers-at-work/)
guest and for his outstanding raise in the application security community as
project leader and valuable contributor.

Reading at Simon's words, it seems to be clear that security awareness in the
development team is the key that lead to build secure software.

> Developers need security training, and the time to make their projects secure.
> Security must be considered as early as possible, and the sooner
> security regression tests are introduced into a project the better.  And
> security people need to get involved much earlier on in projects.  They need
> to talk to developers, understand the pressures they are under and convince
> senior managers that security needs much more time and effort spent on it.

Even this is not possible, it's important to understand that security tests are
not just a kind of performance tests.
 
They engage application logic very deeply and if there is something broken, you
must have the time to fix the issues.

Make your stakeholders aware that security is not buying the Gartner Quadrant
leading tool and some system integration. Security is part of a process that
build your core, the software that runs your business.

Keep them in mind.
