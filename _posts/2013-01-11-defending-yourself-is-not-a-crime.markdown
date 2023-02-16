---
layout: post
title: "Defending yourself is not a crime"
date: 2013-01-11 07:56
comments: true
published: true
featured: false
tags: lets-talk-about simple-life rails activerecord actionpack sql-injection defensive-programming filtering-input waf modsecurity owasp-modsecurity-crs java 0-day cve-2013-0155 cve-2013-0156 cve-2013-0422
thumb: teaser.png
level:
hn:
rd:
---

When I wrote [last week post](http://armoredcode.com/blog/cve-2012-5664-sql-injection-on-rails-dot-dot-dot-again/)
incipt, I wasn't aware I was going to make a prophecy about 2013 and
application security.

But I did it.

<!-- more -->

## CVE-2013 and CVE-2013 and a framework that has bugs

Looking at [RoR](http://rubyonrail.org) community I may feel the sensation that
people are comfortable their framework allows them to cook a web app in
minutes, that it can give them helpers in order to write software the agile way
and that is rock solid: robust and secure.

Please don't misunderstand my point here. I do believe Rails is an outstanding
framework with some advanced security features like:

* html\_safe helpers to filter output
* automagically defense against basic SQL Injection patterns via ActiveRecord
* anti cross site request forgery token

But, like almost every piece of code written by human being, also Rails
framework has bugs and, as consequence, even security ones.

Keeping underlying framework up-to-date is an **critical** activity a security
engineer has to deal with nowadays.
Updates must be:

* possible: you can listen a lot of stories about very customized legacy
  application that they cannot be update anymore since the developers are no
  longer available or since updating some library would break *anything*
* smooth: ideally your updating task would not be longer than a full day of
  work. If it is and your codebase is smaller then the [Linux Kernel](http://kernel.org) than you're missing something

In order to achieve this you have to trust and use your underlying framework
but you don't want to became a slave of its features. This leads to a simple
*mantra* I use everytime I talk to a developer about application security:

> Choose a framework that is security aware but don't rely on that. Your code
> must handle and validate all inputs coming from HTTP request, filesystem,
> database, whatever.

This week two vulnerabilities were found again on ActiveRecord and ActionPack
Rails framework components. A Metasploit exploit was delivered out-of-the-box
exploiting the latter leading to a remote command execution. Panic.

I won't add a bit to precious [Postmodern post](http://ronin-ruby.github.com/blog/2013/01/09/rails-pocs.html) and to the post made
by [metasploit hacker](https://community.rapid7.com/community/metasploit/blog/2013/01/10/exploiting-ruby-on-rails-with-metasploit-cve-2013-0156) deeping those vulns in detail.

Just a recall to update your Rails installation to the latest available and add
some code managing your inputs before using it. You will save a lot of troubles
if you're confident about an attacker can't pass you a malformed value since
you made input validation.

## Java 0-day

Java is fun. There is at least a couple of security advisories per month. 2013
starts with a 0-day [already spotted in the wild](http://malware.dontneedcoffee.com/2013/01/0-day-17u10-spotted-in-while-disable.html)
with a assigned CVE identifier:
[CVE-2013-0422](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2013-0422).

To mitigate this you have only to disable your browser Java plugin, waiting to
Oracle to patch this one.

## Off by one

Don't miss the opportunity to implement a real input validation policy for all
the stuff related to your web experience.
Life is too short to recover from a system compromise just because you didn't
apply some if or regex to make your data safe.
