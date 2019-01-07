---
layout: post
title: "Tales from a login page: intro"
date: 2013-08-08 15:22
comments: true
published: true
featured: false
tags: h4f pentest-with-ruby owasp tales-from-login owasp-top-10 broken-authentication basm login session authentication railsberry 
thumb: authentication.jpg
level:
hn: 
rd: 
---
During 2013 a lot of websites were defaced. Attackers mostly use SQL injection
vulnerable pages to steal data, execute arbitrary commands or make some _nasty
things common people can't understand_

A serious but very underestimate attack entry point is the login page and the
underneath authentication and authorization system.

<!-- more -->

Login page is your web application front door. Everybody in the world reaching
your web site, eventually will deal with it. If you implement poor security
here, your effort spent on security is useless. No secure coding or web
application firewalls with tons of custom rules would help you if your
authentication mechanism is poor.

## SSL and a false friend

When people deal with a login has different opinions about SSL and cryptography.

For Intranet powered web applications, most of developers and sys admins I
faced during my career lays on plain HTTP protocol without any form of
encryption. Please bear in mind that it's very easy to setup an internal
certification authority to digitally sign SSL certificate for your organization
internally deployed web sites. And... it can be be done for free using OpenSSL.

The myth of the Intranet web site is strong also these days. People think
nobody in a LAN segment would eventually launch a tcpdump or friend sniffing
HTTP traffic to steal internal web site credentials.

People from the inside are **not** legal user by definition. There are many
scenarios about internally driven attacks, from people to steal VPN access to
people doing WiFi cracking obtaining an IP address, to people connected to the
LAN that intentionally gather credentials.

Their goals? They depend on the web application it self. For the payroll web
application it can be very interesting to know other people salaries. Other web
applications can disclose classified information, source code or anything it
can be used in a further attack.

For Internet powered web applications I saw HTTPS enabled also on some static
web sites about new startup or new technologies. In this case we can see the
opposite misconception: SSL is not the _panacea_ for web application security.
People think that the closed lock in the browser address bar, it means _your
website is secure_. Don't. It means data is safe from being evesdropped (except
from a Man in the Middle attack, but it's another story).

For short: 

* using SSL is good to protect data confidentiality and integrity
* it must be used either in Intranet than Internet deployed web sites
* it doesn't solve your application security problems

## Information disclosure

Let's say you protected your login form with a digital certificate. Let's say
also your users misspell passwords and they can't login. To avoid frustrated
users, you will warn the users about the username is correct but the password
is not.

This vulnerable login page shows an error message for an existing user Tom,
after an unsuccessfully login attempt.

![]({{site.url}}/images/wrong-password-tom.png)

What about if the username doesn't exist? Should the web application warn the
user about he mispelled his login name?

Of course, the right answer is **not** but there are tons of web site doing the
opposite. They say if the username exists or not.

![]({{site.url}}/images/unknown-user.png)

Let's see from an attacker perspective. Just with some typing I can say if a
user exists or not in the web application database. If that user's password is
weak and if there is not account lockout policy after some failing attempts
evil guys can eventually enumerate existing users and breaking their weak
password.

The idea is that your web application must be very quiet about what went wrong
in authentication step. A laconic message about invalid credentials is a good
choice.

The important bit is that, there is an information disclosure **only if** the
error message is different from the _non existent user_ to _the user exists but
the password is wrong_. 
This is also true if your target doesn't output HTML code but it's a webservice
talking JSON or XML. If the return code is just an OK, KO you don't have
information disclosure.

For short:

* use laconic error message
* implement a lockout policy to disable the account after a large number of
  logon errors in a short time.
* force users to choose strong password to avoid the risk of bruteforce attacks


## Off by one

We will discuss a lot about authentication mechanisms, in this intro post I
just pointed out those two key points you may want to implement in your login
page: SSL and no information disclosure.

Next times, we will talk about sessions and cookies and we will of course try
to break a poor authentication system with a ruby code.

Enjoy it!
