---
layout: post
title: "Untold: Owasp Orizon is died and I'm sad of it"
date: 2012-11-20 07:54
comments: true
published: true
featured: false
tags: owasp owasp-orizon builders core-review sast antlr java opensource github milk storytelling
thumb: seaside-horizon.png
level:
hn: 
rd: 
---

In 2006 I started an ambitious project, an opensource SAST engine built in Java
I called [Owasp Orizon](http://www.owasp.org/index.php/GPC_Project_Details/OWASP_Orizon_Project).

Of course the name was intended to be _horizon_ but I mispelled the word and I
found silly to cover my lexical mistake.

After 3 years without no fresh code, I think the project can be considered
closed, also if some people seem to be interested into it.

<!-- more -->

![]({{site.url}}/images/owasp-orizon.png)

It was yesterday and I found two emails about Owasp Orizon in my 
[owasp.org account](mailto:thesp0nge@owasp.org). In a first email an American
guy asked how he can help me in improving the tool and in a second email a guy
from I guess Asia region asked about [milk](http://milk.sf.net) a java
interface for orizon I wrote in 2006.

True to be told Milk is even more abandoned than Orizon. When I answered the
latter about he should use orizon instead, he said that even this one seems not
to be updated since 2009.

The fact is that I had no fresh energy to put in this project anymore. I
switched my programming interest from Java to Ruby and I'm working on
[sake](https://github.com/codesake/sake) now that it will become an hybrid
security testing tool and the engine of an application security startup,
[codesake](http://codesake.com).

## The concept

The first idea I had in mind when I designed Owasp Orizon was to translate the
source code under security review in an intermediate XML representation and
making security checks over it.

If you make your security engine to work on an XML syntax, supporting a new
programming language is just writing a translator to that XML representation.
Of course you will add some specific security check to the knowledge base.

This approach makes sense to me even today but it doesn't work (or at least I
wasn't able to implement it in a working way).

Translating from different languages to a single XML syntax makes a lot of
specific language expression power to be lost. 

Java has a very complex subclassing mechanism. There are interfaces that they
can be used to design a common API interface in example, there are abstract
classes, there are the common hierarchy system in which a class can extend one
or more classes.

Ruby has a completely different approach to modularity. Translating the twos in
a common XML will result in a hard to read XML node full of options, attributes
and so on.

## The boost 

We were in 2008. I got married and Owasp Orizon has one of its most prolific
moment. I had the opportunity to fly in New York city for Owasp AppSec 2008
telling the world about the piece of code I was writing.

Feedbacks were good and I had two job opportunities today I regret I declined.

## The fall

But in 2009 I had some harsh emails from a guy contributing to the project
saying I wasn't able to listen and to attract people.

A lot of people, from Italy I had to be honest, came here and say "I want to
help" and after some mail they disappeared. Eventually I discovered they used
Owasp Orizon as self-promoting during consultancy presales.

I loose motivation and energy. The code was ugly and writing an antlr parser
instead of XML translation was really hard, we eventually moved to freecc but
it doesn't last.

Owasp Orizon source code was not updated anymore and you can imagine the whole
story.

## Something I learnt

Being part of Owasp Orizon project was inspiring and I learnt a lot both in
terms of coding than in terms of community. 

I traveled a lot and I started taking public speeches. 

I enjoyed being part of [Owasp](http://www.owasp.org).

But it's time to move on. It's time to improve and become bigger and bigger.

Enjoy It!

_Winter sea side horizon picture is by [me](http://www.flickr.com/photos/thesp0nge)_ 
