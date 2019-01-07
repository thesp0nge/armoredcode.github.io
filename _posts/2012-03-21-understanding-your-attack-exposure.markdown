---
layout: post
title: "Understanding your attack exposure"
date: 2012-03-21 9:00
comments: true
published: true
featured: false
tags: breakers pentest attack api awareness pentest-with-ruby
hn: http://news.ycombinator.com/item?id=3734208
rd: http://www.reddit.com/r/netsec/comments/rcs59/understanding_your_attack_exposure/
---

## You see an HTML form, I see your database

Today, we will brefly talk about what do you expose when you publish a web
application and what do an attacker see when you issue your deploy command.

First mistake people do when creating webapps is to think that only _good guys_
will come and use your services. **No**, people will try to mangle your URL
parameter values seeing how the application will react. People will try to
submit code to see if do you escape it or not and even more People will try to
pass SQL commands to see if injections are possible.

<!-- more -->

You may ask why, but this is not relevant. People will do this. Of course, if
someone starting from just poking around your website will try to break
crafting the attack pattern better and better they will make a crime. No fuss,
so... guys... don't try this at home, or at least make it to some friend
website for suggestions and awareness purposes only.

## Think your code to be secure and consistent

Let us imagine that a person is knocking at your office door asking _'I have to tell SEO guy the URLs our application has for building sitemap, can you help us?'_

![](http://www.history.navy.mil/photos/images/g30000/g32750.jpg)

If the answer is either _No_ or _I'll take a couple of days to make it_, than
you **have** a problem.

Despite about how big is your site, if you don't know to build quickly a
sitemap of what are you exposing, where are the input forms page, then you
don't control your attack surface. Maybe your code is too big and so you won't
be able to identify place to make mitigation steps or maybe you didn't write
enough documentation, the real issue is that an attacker is a step in front of
you and he will look like for the following. 

Attackers need to submit input in order to to exploit your code so the necessary looks for:

* web forms
* parameters in query strings (search keys, actions flag, post or item identifiers...)
* items in HTTP requests if they are actively used in HTML output generation (referrer, ...)

![](http://upload.wikimedia.org/wikipedia/en/f/f9/Death_star1.png)

As developer you must take care about what are you taking as input from the
network (either the Internet or just the intranet it doesn't matter, most
exploiting are from the inside nowadays).

You can use the following checklist for every single API exposing services:

* Do I filtered POST parameters using [Owasp ESAPI](https://www.owasp.org/index.php/Category:OWASP_Enterprise_Security_API) or something similiar?
* Do I filter all parameters from query string?
* Do I use any item in HTTP request? If yes, I filter all of them?

Your API **must** be robust by design enforcing a filter policy for every single input you take from the outside.

## A small recap

Since you're a developer you don't have an attacker mindeset. For you a form or
a parameter is just data to be submitted to your application. And this is fine.

The important thing here is that you must move into the _dark side_ and
thinking about your API endpoint to be sure that no tainted data will be passed
to your backend.

## [Out of Topic announcement] - Blog will be paused for two weeks

Guys, I'll be out for some very personal issues and I won't publish posts at least since April (not an April fool however :().

So please be patient and next month we will talk about newest hacking strategies, news in the application security space and other stuff.

Next posts in pipeline will cover:

* Understanding your risk: a small recap over the [Owasp Top 10](https://www.owasp.org/index.php/Top_10_2010) as companion for this post;
* Design a strong API using [Sinatra](http://www.sinatrarb.com) - part 1: a small tour in writing an API application using Sinatra web framework.
