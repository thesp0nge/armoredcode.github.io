---
layout: post
title: "Creating awereness on an hostile environment"
date: 2013-03-12 07:57
comments: true
published: true
featured: false
tags: awareness railsberry2013 talk survey builders appsec
thumb: hostile.png
level:
hn: https://news.ycombinator.com/item?id=5360998
rd:
---

With a colleague we were wondering about how much difficult is to create an
application security awareness climate in big corporate development team.
Please bear in mind that since I'm working in Italy my experience is very
narrowed to my country. If you have different stories to tell, please drop them
in this post comments area and share them.

_Trying to make people aware about security risks they occur writing unsafe
code, will make yourself a friend or foe?_

<!-- more -->

## The harsh part of the story

Fact: software (most of it) is **not** secure and neither servers' daemons nor
servers' configuration is focused on security.

Fact: small teams (startups, small web agencies) actually do have developers
super stars. Some of them knows something about appsec but the time and the
market are their enemies. No time and no budget for security tests.

Fact: large teams (corporate) may have some skilled developers. Security is
seen as _yet another compliance and boring test we are supposed to pass_. _If
our code won't pass security checks, we will rely on firewalls, we don't want
to spend our precious time in fixing the code for something is not a bug._

Fact: security costs. We assume you use a freelance application security expert
to engage your web application. The tests will cost you money either in terms
of freelance fee, than in terms of developers' time to mitigate
vulnerabilities.

Fact: ignoring security costs in your budget will expose you to larger costs to
remediate a security breach in your database. Either in terms of fixing up the
code, than in refunding hungry customers. You may want also to consider the
costs you pay to build a trust relationship between you and your customers
during the startup period.

5 facts to draw the application security perimeter. When you will deploy any
awareness program into a big organization, you will fight at least those
misconceptions. All of them are leading to a pure and cost saving concept: I
don't need security because it takes me money so I can ignore risks.

## An hostile marketplace

Most of developers don't trust security specialists. At least in Italy there is
the strong misconception that an application security specialist is not able to
code and that a developer is not able to understand security topics. Of course
both of those sentences are wrong (with some remarkable exceptions I won't
discuss here) but the truth part is that finding a meeting point for those two
worlds (development and security) is a compelling task.

As a startupper creating a trustworthy relationship with their customer, an
application security specialist will be able to spend years in creating a
realation ship with a development team. It must be honest, competent and giving
the team something valuable to work over.

A security report saying "Fix this XSS because it's security saying this" is
useless. You have to carefully explain what's wrong with that code and how to
mitigate. They will be create software better than you will (most of times) but
you have to start talking to developers using source code if you want to create
a solid connection.

Creating awareness about security risks and make other people to trust you, it
is a process taking years in order to be successful. In my experience, you will
migrate from a not security aware development team to a Secure Software
Development Lifecycle ready team in 5 or 6 years.

During this period you will:

* create secure coding guidelines and you will listen developers criticisms
  about them and you will iterate over and over
* make them pleased to ask you for a web application penetration test **and**
  for code reviews. You will provide valuable reports containing ideas not
  impositions
* meet developers and train them about secure coding and about new attacks

It's impossible to complete this task without any programming background.

> You can't make code reviews or saying what's wrong in other people code unless
> you wrote code yourself as well. Every application security specialist must
> write code and have worked with strict deadlines in order to understand how to
> talk to developers.

Look at this very basic C snippet of code.

``` c a reading experience
int fd;
char **buf = malloc(8192);

fd = open("myfile", O_RDWR);
// some instructions here
read(fd, buf, 8192);
// other instructions there

close(fd);
```

Imagine you never wrote a single line of C code in your life. How can your code
review be useful in this scenario? Are you a good security specialist just
because you ran
[rats](http://www.softpedia.com/get/Security/Security-Related/RATS.shtml)?

In my opinion you must be honest and saying you can make code review only if
you really can write it better:

``` c a secure reading experience
#include <sys/file.h>
#include <stdio.h>
#define BUF_SIZE 8192

int fd;
char **buf = malloc(BUF_SIZE);

fd = open("myfile", O_RDWR);
if (fd == -1) {
  perror("myapp");
  exit(-1);
}
if (flock(fd, LOCK_EX) == -1) {
  perror("myapp");
  exit(-1);
}

// you can now make something here without being worried about TOCTOU
// when you read... remember to leave a char for the end of line...
if (read(fd, buf, BUF_SIZE-1) == 1) {
  perror("myapp");
  exit(-1);
}

if (close(fd) == -1) {
  perror("myapp");
  exit(-1);
}
```

_this example is taken from a real code review I made a couple of week ago.
Developers actually don't care about TOCTOU because the read call was too much
call to the open to justify the lock... creating awareness is also force
yourself not to laugh so hard._

## Tell me your now

I need your voice now... I create a
[survey](http://www.surveymonkey.com/s/8RG523F) to ask people their position
about web application penetration test. I will use as slide number 2 or 3 in my
upcoming [Railsberry 2013 talk](http://www.railsberry.com).

Q: _Why you don't make any web application penetration test when I deploy a new
web application (or a new feature)?_

* You're wrong. I do make a web application penetration test when I deploy
a new web application or a new feature
* No budget. Security costs are too high for us, we're a startup and we're
focused on business first
* No need to. We're a big development team. Our code is robust and strong.
We won't occur in any security incident. Ever
* No time. We are missing our deadlines. We don't have time to spent in
security tests. We are safe from risks. We have firewalls.
* I don't care. Seriously, security is a word spent by sales men to sell
antivirus or similiar stuff. I don't think my web application will be attacked
by "so called" hackers.

What's your positition? Do you ask a security guy to make a penetration test
over your code?

_Image curtesy by [Glyn Freeman](http://www.swanseacameraclub.co.uk/gallery2011/Glyn%20Freeman/album/slides/HOSTILE%20ENVIRONMENT.html)_
