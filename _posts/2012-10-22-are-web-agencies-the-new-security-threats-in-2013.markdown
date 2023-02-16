---
layout: post
title: "Are web agencies the new security threats in 2013?"
date: 2012-10-22 07:50
comments: true
published: true
featured: false
tags: builders outsourcing web-agency appsec ssdlc
thumb: web-agency.jpg
hn:
rd:
---

An economical crisis time has been started 4 years ago and this eventually
changed how people engage contractors to develop code.

At least in Italy, most of time big companies outsources some services like
communication driven website, forgetting about their overall security level is
the one of the weakest link.

<!-- more -->

A month ago Italian [BNL bank](http://www.bnl.it) was attacked and some
technical people credentials were disclosed on [hackernews](http://news.thehackernews.com/101).
The breach was found in a secondary website that was lying almost forgotten in
the cyberspace.
Disclosed credentials were eventually still valid since the website was no more
update but it was still used.

> Clueless manager would say he's going not to pay someone to update an old
> crappy code because a full migration project will drive too much money away
> from real core business. Most of time, real core business is communication
> those days.

True to be told investing a large percentage of money in communication and
marketing it makes sense but not applying a vulnerability management policy
even for outsourced, not corporate websites it is a **real risky** decision.

Let me be clear at this one. Sometimes it's hard to clearly distinguish a good
strategical path to follow, however you must keep in your roadmap all your
source code to at least a per year check and bugfix session.

## How to engage an outsourcer for a website

Let's make this assumption. Your organization have a sort of information
security team that it is able to make at least a tool driven automatic web
application penetration test. No matter the tool you bought.

You need to develop a secondary website eventually with some sort of HTML
forms, let's say for search for website contents and for keep in touch with
your business.

In the [very beginning of this blog](http://armoredcode.com/blog/understanding-your-attack-exposure/)
we discuss about the web forms as part of your website attack surface. You hit
the point, having a non full static website, makes you vulnerable to web driven
attacks.

You google some webagency website, looking fot the one with the compelling
portfolio and you ask them for a quote.
At this point in most cases, you sign a contract with them giving a sort of
impossible to reach deadline and you put your website online a week or two
later.

**Wrong decision.** A development team with no team won't perform tests so your
website will be eventually full of bugs even before spotting for security
issues.

A clever approach can be:

* give your outsourcing IT team internally adopt secure coding guidelines for
  the language they will use and for operating system, database and web server
  configuration hardening;
* ask your contractor to allow your team to make:
  * a vulnerability assessment over the machine that it will host your website;
  * a web application penetration test over the release candidate version. This
    will make sure you'll test code that it's almost ready to go in production
    buy it will leave some days for the contractor to fix security issues;
  * a codereview over the source code to make sure it will hit security KPIs
    you give when both parties signed the contract
* specify in the contract that you own the source code and they have to give
  you access to their source code repository

If you don't have an internal security team that it is able to make security
checks you have to find a security contractor that makes the tests for you.
It's important that you drive the activity, not giving the task to the web
agency.

## What to provide to your webagency

So, when you sign the contract you give the following to your webagency:

* a copy of your internal secure coding guideline for the language the
  contractor will use. If you don't have such a guideline, you **do** need to
  engage a security contractor to write you one.
* a copy of hardening guidelines for operating system, database server,
  application server. You can refer to [standard guidelines](http://www.cisecurity.org/)
  but eventually it's a good idea develop something custom in your IT.

## What to ask to your webagency

In the contract you have to:

* ask for KPIs about the security level for the code they will write
* make sure they will grant the ability to make all the security tests you need
  accordingly to the laws in use about releasing you of responsibility about
  the attacking attempts you will conduit

Don't underestimate the risks you will face if you outsource web development
without caring about security.
Attackers won't knock your front door but they sometimes will try to force the
service door instead.

Enjoy it!

_Image by [bananasontoast](http://www.flickr.com/photos/bananasontoast/4825924954/)_
