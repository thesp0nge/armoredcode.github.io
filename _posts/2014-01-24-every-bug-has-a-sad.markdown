---
layout: post
title: "Every bug has a sad, sad song"
date: 2014-01-24 08:01
comments: true
published: true
featured: false
tags: builders breakers c appsec discussion bug testing tdd owasp
thumb: cowboy.png
level:
hn:
rd:
---

It was a busy month. Web sites out there are still attacked by villains and the
first
[Codesake::Dawn](http://dawn.codesake.com/blog/announce-codesake-dawn-v1-0-0-released)
major release was out this week. That's because I didn't post anything since
last December.

Today, I want to share a consideration coming out from a discussion I had a couple of days ago:
> Bugs are by definition security issues.

Do you agree with that? I don't, and let's see why.

<!-- more -->

## Developers are clueless

There is a common misconception in the (Italian ?) Information Technology
world. Developers are seen as a bunch of lazy people, unwilling to learn about
security or safe coding, putting online bad code just for sake of release
something.

Well, majority of developers I reached all day (since they are friends of mine
or by chatting on forums or on twitter) do care about application security.
They want to know more about how to deal with SQL Injections or Cross Site
Scripting, but **no one** is able to explain the security risk in terms of
code.

And this is the major point. There is still a gap between developers and
security people because most of the latters (just in Italy?) speak in
_penetration testing_ terms instead of showing how to solve the security issue
in _put a popular programming language you like most here_.

## But application security specialists are even more

Again, it's useless to tell a development team leader (who have strict and
sometimes nonsense deadlines from the management), for a Ruby on Rails or .NET
web application:

> This is my _OWASP or something else_ compliant report that tells that your code is suffering from Cross Site Scriting.
> Attached there is a Java snippet of code telling how to safetly deal with your code.
> Please fix it.

This is **nonsense** for a couple of reason:

* you (appsec guy) didn't use the same programming language the vulnerable web
  application is written to. It's like explaining using Spanish, that a French
  person are using phrasal verbs the wrong way.
* you (appsec guy) just copy cat an example but safe snippet of code taken from
  who knows where, pretending a team of developers to understand it and putting
  that code in their context the right way. This is completly a wrong approach
  to application security.

Application security people, when they present a security issue to a developer,
they must explain how to fix it using the application code providing a very
primitive patch. Developers are there to improve the patch, but if you provide
a copycat snippet of code in a random programming language, there are no chance
people will listen at you. Period.

## Bugs

That's why when during a discussion a security man said _"a bug is by
definition a security issue"_ I raised a red flag. Bugs are bugs, they are
mistakes that can make a software to behave in a different way than its specs.
Of course one or more bugs can introduce under some circumstances a security
issue but a direct correlation can be, of course, proven.

In the discussion then I wrote a piece of code like that, asking for the
exploiting code but I had no answer for that guy:

``` c a sully programming mistake: starting a loop from 1 instead from 0
#include <stdio.h>
#include <stdlib.h>

#define   MAX_LENGTH  4096

void do_something(char *c) {
  strncpy(c, "antani", MAX_LENGTH - 1);
  return;
}

int main(int argc, char **argv) {
  // my specs are to iterate do_something routine 1000 times
  for (int i=1; i<1000; i++) {
    char *buf = malloc(MAX_LENGTH * sizeof(char));
    do_something(buf);
    free(buf);
  }
  return 0;
}
```

## Off by one

Which is the point?

We (application security specialist) must take care about writing a security
report the right way. We don't have to complain about bugs telling developers
are not able to do their job... we must make sure we are able to explain them
how to do their job in a secure way.

And still... a bug is a bug, it doesn't involve being a security issue by default.

Enjoy it!
