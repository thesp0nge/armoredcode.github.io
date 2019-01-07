---
layout: post
title: "Understand your risk: disclosing information"
date: 2012-04-17 10:00
comments: true
published: true
tags: builders breakers pentest webapp vulnerabilities
hn: 
rd: http://www.reddit.com/r/programming/comments/sdydd/understand_your_risk_disclosing_information/
---

Few things are dangerous like giving attacker detailed information about how
your application works and how it can be subverted.

Your application uses error messages to talk to users let's see the best
strategies to save the day from unexpected access.

<!-- more -->

## The architecture behind

Make smart decisions here. You have to choose great backend technologies you're confident to work with. 
Choose the operating system you are trained to setup and customize, you can
also ask some friend or even use some cloud computing services such as
[heroku](http://www.heroku.com) to manage your backend.

The important bit here is that you do know the technology, so you can make
strategic decision based upon something you know.

> Don't choose Ruby on Rails because it's cool in the startup world in the
> Valley, choose it because you like Ruby syntax, you are trained to test your
> code and you love it like MVC framework.

Configuring your applciation server here is crucial. It's not a matter of
hiding the server or the version, it's about harden your installation and lock
it down.

Consider [this](http://armoredcode.com) blog. It runs on a [nginx v1.0.14](http://nginx.org/) 
on a ubuntu linux installation over [linode](http://www.linode.com/).

Blogging technology is based on [octopress](http://octopress.org/) so the site
is statically generated, no need for DB just HTML, javascripts and other SaaS
APIs.

Disclosing all those information doesn't scare me at all. I'm confident in
managing my installations instead of trying to hide details from banners.

## Make a strong API is the best shield you can have

It's true, there's nothing like having a bullet proof rock solid API driving
your backend to give you a strong feeling about your applicatin won't be easily
compromised by an attacker.

Some rules when you think an API:

1. use [REST](http://en.wikipedia.org/wiki/Representational_state_transfer) and
   [CRUD](http://en.wikipedia.org/wiki/Create,_read,_update_and_delete)
2. think that people will abuse it. If you create a 30 characters length field
   in your database, double check the size the user submit. First attempt an
   attacker could make is to see how your code behaves when triggered with more
   data than expected
3. give detailed error messages only to developers for fixing, not to regular
   users. 

   Consider the following example. [Github](http://github.com) ![](/images/github-failed-login.png)
   gives you a laconic error message that says that nor the login name nor the
   password is correct and so you can't login. This is very important to avoid
   attackers to try to brute force a password after guessing the username is
   correct.

   A bad written code gives you detail about what's went wrong. Messages like
   "Username unknown" or "Mispelled password" give an information to an attacker
   that something she inputs were correct and therefore she can try a bruteforce
   attack against passwords.

> It's better not to hide yourself to achieve security, but not exposing too
> much information is also good.

4. manage errors and exception in a good way. You must see the exception the
   code you use can raise and catch them to make sure to control also
   unexpected situations.

## Don't trust on SSL

There is a category of project managers or even software developers that says:
_"I use SSL so my website is secure, I don't need more effort to spend"_

Those people are lying to themselves. SSL protects the communication channel
and data from integrity and repudiation. SSL has nothing to do with safe coding
and disclosing error trace.

Keep this in mind when you put online your code.
