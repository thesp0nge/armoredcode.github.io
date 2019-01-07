---
layout: post
title: "The fragile Internet"
date: 2012-11-06 07:55
comments: true
published: true
featured: false
tags: breakers anonymous nbc 5-11 defacement xss owasp 
thumb: fragile.png
level:
hn: 
rd: 
---

It was a yesterday's news that [anonymous](https://twitter.com/YourAnonNews)
and other cracker's crews attacked and defaced large number of corporate
websites.

[November 5](http://www.wired.com/threatlevel/2011/11/remember-remember-anonymous-celebrates-the-5th-of-november/)
it is a very symbolic data in the anonymous underworld and a massive defacement
attack was carry on, at least, against [PayPal](https://www.paypal.com),
[Symantec](http://www.symantec.com) and [Telecom Italia](http://www.telecomitalia.com) 

Anonymous and other crews activities tell us an old story: the Internet is
fragile and your web applications can be attacked anytime, anywhere and most of
them are breakable.

<!-- more -->

## The same old refrain

It was 10pm when I had dinner last night. My wife and my son were sleeping and
I checked [https://twitter.com/thesp0nge](my twitter stream).

Some friends were talking about a massive defacement activities carried on by
anonymous hactivists and other cracker's crews not connected to the former.

It wasn't the first time both [PayPal](https://www.paypal.com) and
[Symantec](http://www.symantec.com) were attacked. The latter suffered from a
[source code leakage](http://www.zdnet.com/symantec-source-code-leaked-on-pirate-bay-7000004765/)
some months ago.

The news that impressed me much was the attack [against Telecom Italia](http://thehackernews.com/2012/11/anonymous-hack-30000-accounts-and.html).
In the news it's reported that attackers found more than **3.000 Cross site
scripting vulnerabilities**.

Even Owasp WebGoat web application has less XSS.

Of course, this is not the only hole they exploit. The report talks about
poorly written *.htaccess file* and *weak passwords* that they lead to a
successful attack.

## The power of now

In a post about [web agencies](http://armoredcode.com/blog/are-web-agencies-the-new-security-threats-in-2013/) 
and about [marketing driven choices](http://armoredcode.com/blog/border-line-between-marketing-and-security-features/) 
I talked about the dangers of publishing a web application without a security program.

Marketing departments want to deploy new websites, new features, new dynamic
content to promote goods and to increase business. This is completely fair but
it can't be done without security awareness. The problem is that they don't
have any clue about their websites can be attacked and sometimes they didn't
trust their security departments trying to make them aware.

If your web manager says **the website has to be online now** or even worse
**we must add this brand new feature ASAP**, you must take care about the new
content can't be exploited on the wild and you have to make all the necessary
security tests before the content has to go online.

## Off by one

Yesterday's [anonymous attack](http://www.examiner.com/article/anonymous-nbc-hacked-defaced-honor-of-guy-fawkes-day) makes me think about how hard is our work.
Sometime people says that only banks deserve to be protected. But attacking
[broadcasting companies](http://thehackernews.com/2012/11/nbc-websites-hacked-to-promote-nov5th.html)
or
[telcos](http://thehackernews.com/2012/11/anonymous-hack-30000-accounts-and.html)
can amplify your activism claims tons of times.

Don't trust Web Application Firewalls. They will help but clever attackers may
override their rules. 
Force developers to write secure code instead.

Enjoy it!
