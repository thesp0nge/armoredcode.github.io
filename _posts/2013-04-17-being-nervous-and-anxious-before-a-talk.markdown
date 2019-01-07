---
layout: post
title: "Being nervous and anxious before a talk"
date: 2013-04-17 08:16
comments: true
published: true
featured: false
tags: appsec ruby pentest-with-ruby codesake railsberry railsberry2013 talk self simple-life
thumb: anxious.png
level:
hn: 
rd: 
---
It happens all the time I have to deliver a talk. Some days before my
anxiety-meter level goes out of scale.

It will last until slide number 4 when I will recall that all the attack stuff
I will show during the speech are not intended to be used by offense.

<!-- more -->

## Point of singularity
Last time I had a public talk was last june for [Italian Ruby Day](http://rubyday.it) 
and I finished slides during the talk right before mine.

In the past I delivered talks for [Owasp events](http://www.owasp.org) in
Milan, Ghent, New York city and Cracow too and I finished slides the same day
of the talk or the evening right before.

For [sikurezza.org](http://sikurezza.org) I delivered talks in Milan and Padua
for SMAU and a very good technical conference called WebbIT (now disappeared)
and the same... slides were never ready in time.

I'm scared... for upcoming [railsberry 2013 talk](http://railsberry.com) slides
are ready and video with attack demos too.
_(I lied... I had to find a pretty decent Dr. Evil image in HD for slide number
5 when I will recall to all outstanding developers in the room that they have
to turn themselves in attackers for awhile)_

## Some stats
40 slides... for a 30 minutes long talk. Not all of them has content, some are
titles or section separators so I can play well being in 2 minutes per slide in
average.

I've got 8 slides with ruby code and three videos showing how do I use this
ruby code I wrote in real world attacks under a broken web application I wrote
for this conference.
The broken web application is already available on 
[my github](https://github.com/thesp0nge/railsberry2013). 

## What I will be talk about
The [broken web application](https://github.com/thesp0nge/railsberry2013) is build with three security flaws in mind:

* the authentication mechanism gives too much information to the user failing
  the login. It says that it doesn't know the user if it's not found in the
  database and it gives a different error message if a valid user mispelled the
  password. I will show how to exploit it enumerating good user of that
  application. **Please note:** you do need know a valid username to try guessing
  (because you have to simulate a good user that mispells the password.
  Owasp testing guide test for bruteforce attack is
  [OWASP-AT-004](https://www.owasp.org/index.php/Testing_for_Brute_Force_\(OWASP-AT-004\))
* the /hello API take a _name_ parameter saying hello to the user not filtering
  the input... so it's vulnerable to a reflected cross site scripting.
  The /login API too is vulnerable to reflected cross site scripting since it
  uses the login name the user just submitted as output without filtering.
  I'll show how to exploit those two vulnerabilities.
  We will implement test
  [OWASP-DV-001](https://www.owasp.org/index.php/Testing_for_Reflected_Cross_site_scripting_\(OWASP-DV-001\))
  as the latest Owasp Testing Guide.
* the robots.txt has some disallowed entries but one of them is missing some
  checks about preventing unauthenticated people to access it. I'll show how to
  check for open URLs found in a robots.txt file with a single command.
  Owasp testing guide test is
  [OWASP-IG-001](https://www.owasp.org/index.php/Testing:_Spiders,_Robots,_and_Crawlers_\(OWASP-IG-001\))
  here.

Videos will be available on armoredcode youtube channel right after the event
and slides will be available also too on slideshare and speakerdeck. Source
code is almost already available on [codesake github archive](https://github.com/codesake). 
I had to rebeand my previous released cross gem.

Enjoy it!
