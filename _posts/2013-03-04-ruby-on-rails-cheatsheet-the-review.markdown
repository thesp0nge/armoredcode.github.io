---
layout: post
title: "Ruby on Rails cheatsheet: the review"
date: 2013-03-05 09:50
comments: true
published: true
featured: false
tags: owasp builders ruby ruby-on-rails cheatsheet appsec
thumb: cheatsheet.png
level:
hn: 
rd: 
---

[Jim Manico](http://www.manico.net/) is a friend and a rinomated security
professional. He
[announced](http://lists.owasp.org/pipermail/owasp-leaders/2013-February/008735.html)
in [Owasp](http://www.owasp.org) mailing list that a [Ruby on Rails cheatsheet](https://www.owasp.org/index.php/Ruby_on_Rails_Cheatsheet) is
available.

<!-- more -->

I asked Jim to introduce himself.

{% blockquote Jim Manico http://www.manico.net %}
Jim Manico is the VP of Security Architecture for WhiteHat
Security, a web security firm. He authors and delivers developer
security awareness training for WhiteHat Security and has a background
as a software developer and architect. Jim is also a global board member
for the OWASP foundation. He manages and participates in several OWASP
projects, including the OWASP cheat sheet series and the OWASP podcast
series.
{% endblockquote %}

This is a critical review about the cheatsheet. My concern, as well other
venerable Owasp leaders, is to provide content that it can be consumed by
developers rathern than only by security researchers with some coding skills.

The cheatsheet is authored by:

* Matt Konda - mkonda [at] jemurai.com
* Neil Matatall neil [at] matatall.com
* Ken Johnson cktricky [at] gmail.com
* Justin Collins justin [at] presidentbeef.com
* Jon Rose - jrose400 [at] gmail.com
* Lance Vaughn - lance [at] cabforward.com
* Jon Claudius - jonathan.claudius [at] gmail.com
* Aaron Bedra aaron [at] aaronbedra.com

Kudos for all those guys for their work and commitment.

## The review

First of all, let me say one thing: the document Jim and other guys wrote is:

* clear
* with security in mind
* almost practical

This cheatsheet has very valuable content in it and you can use it, **right
now**, to improve your Ruby on Rails secure coding workflow.

The KISS paradigm is applied to this document and every item is shortly
described giving the reader the opportunity of going deeper with further
investigations.

The first thing I found it would be better in terms of approach is that there
is no a logical organization for the item described in the cheatsheet. It would
better having them grouped by web application macro areas like _authentication
& authorization_, _input validation_, _error handling_ and stuff like that.

[Command injection](https://www.owasp.org/index.php/Ruby_on_Rails_Cheatsheet#Command_Injection)
wrote this way it's not a Rails specific issue. Of course the important bit is
not to bind user controlled inputs to strings passed to the operating system as
commands to be executed.

The [part about authentication](https://www.owasp.org/index.php/Ruby_on_Rails_Cheatsheet#Authentication)
suggests about using Devise (and this is good) but in a cheatsheet there is no
enough space to cover how to create an autentication mechanism using Devise.
True to be told the code snippets provided are not enough for an early reader
knowing nothing about implementing a good authentication mechanism. You may
want to read a blog post I wrote in November about [implementing an authentication mechanism for Padrino and omniauth](http://armoredcode.com/blog/crafting-an-authentication-subsystem-that-rocks-for-your-padrino-application-with-omniauth/)
It would be a better approach to grab a list of authentication frameworks, with
pointers and blog posts, and give the reader some security tips for each of
them.

For the [forceful browsing](https://www.owasp.org/index.php/Ruby_on_Rails_Cheatsheet#Insecure_Direct_Object_Reference_or_Forceful_Browsing)
section, an example about how to use cancan to enforce some authorization
controls.

Finding [business logic bugs](https://www.owasp.org/index.php/Ruby_on_Rails_Cheatsheet#Business_Logic_Bugs)
is a tricky task, the most difficult in a web application penetration test. As
a security tester one of your task is trying to subvert application business
logic in order to have the target application to do whatever you would do. And
the penetration test is missing from the key ways to protect your application
from business logic flaws.

## We love

* Upcoming Rails 4 is cited
* Rails is growing up as web framework and it's great having security material
  to create awareness

## We love less

* The overall document is for sure intented to be consumed by developers but
  it's clear it has been written by security researchers
* Pointers to Owasp attack detailed description must be placed in order to
  create awareness about risks. My experience is that a developer is not
  trained to see security risk so he couldn't understand at first glance why he
  should change his working code.
* Covers only ActiveRecord as ORM

## Off by one

Of course, the
[authors](https://www.owasp.org/index.php/Ruby_on_Rails_Cheatsheet#Authors_and_Primary_Editors)
work is valuable and it's pretty easy to use this cheatsheet in everyday
workflow as a reference. I'd like to see it improved with other Ruby powered
web frameworks like Sinatra or Padrino and for other ORMs like Datamapper.

This cheatsheet it's a draft as authors say in the [web page](https://www.owasp.org/index.php/Ruby_on_Rails_Cheatsheet), so I can't wait to see any great improvements, hopefully more about upcoming Rails 4.
