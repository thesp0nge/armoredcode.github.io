---
layout: post
title: "Is Design by contract the solution for safe coding?"
date: 2012-05-10 08:02
comments: true
published: true
featured: false
tags: builders programming-language design development api summer-project coat design-by-contract
hn: http://news.ycombinator.com/item?id=3952317
rd: http://redd.it/tg682
---

## A long time ago, in a University far away... 

It was fall 1999 when I first met [Eiffel](http://www.eiffel.com) programming
language and the [Design by contract](http://en.wikipedia.org/wiki/Design_by_contract) concept it introduced.

Mixing _pre-conditions_, _invariants_ and _post-condition_, an Eiffel class
creates a sort of contract with other classes on about its behaviour, the data it can
deal with and which kind of output you may expect.

As an aside, universities here in Italy are great but there is too much theory
and very few applications of what you have just learnt. 

When teacher introduced a formerly unknown programming language, no used in the
"real"<sup>1</sup> world I got the blues. I thought those was yet another
useless thing I had to study to pass my exams but I won't use it anymore in the
"real" world.

What an idiot.

<!-- more -->

## Contract leads to knowledge that leads to be deterministic

As an appsec specialist, sometimes I asked developers about some parameters
values they use in their pages just to figure it out the accepted ranges to
lockdown WAF configuration.

Sometimes they can't answer me.

The problem is that when a piece of software is written and maintained from
different people over the years without any written form of documentation, it
is rather impossible to figure it out if a certain parameter passed by POST,
let's say, can be tampered or it must be less than 100 bytes or it must be
fullfill a regular expression, without looking at the code.

Make a reverse engineering process just to figure it out what a class is
expecting as input is:

1. time consuming
2. complex and sometime impossible if the original developers wasn't in the
   team anymore
3. insecure: with a non precise answer you have to **figure it out** that led
   to interpretation that is personal and that it is subjective.

With a design by contract approach, your code will explicitly expose what it is
expecting in input for each public method and which is the output you have to
expect.  

So answering the question on which kind of data is expecting as input is
trivial now. You have just to look at your method definition and you have it.

But, what about if a contract won't be honored?

## The security pitfall of DoC

You've got it. Saying what a method is expecting doesn't mean other people will
play fair and they will honor the contract. Design by contract doesn't say
anything about this.

More precisely, DoC says that it's not up to the class declaring a contract to
check if this one is fullfilled or not.

This is somewhat annoying. You introduced a great infrastructure that says how
to use your API but you don't make your code in charge of self defending.

## Let's save something from the design by contract approach

In my opinion, the idea behind DoC is **great**. It forces you to design and
document a class writing pre-conditions and post-conditions.

This is very good since you can't skip writing documentation for your methods.

## Merging the stuff

Behavior Driven Development says that you must start from test cases desecribing
how that class has to behave and then code your methods to let the tests to
pass. It doesn't say anything about contraints for methods, let's say BDD just
make sure you develop a method that fullfill your post-conditions.

DoC says that you must place rules even in pre-conditions to declare other
classes which kind of data your method is fine to deal with.

Let's say that using assertions in your code you can enrich it by adding some
runtime checks about the contract conditions. But this is up to you, it's not
part of the contract, is an enforcement you must place to make your code
self-defending.

> It would be great having something that starting from the BDD-ready test
> case, it would create a class skeleton with pre-conditions, post-conditions,
> pre-built security assertions and invariants.

## My summer project: Coat (COntract And Test)
Starting from [Marc-André](http://macournoyer.com/)
[work](http://createyourproglang.com/) I'd like to write a sort of metalanguage
that recreates this workflow adding contracts and enforcement rules to the
class to make sure it would respect the contral even if under attack.

I'm not thinking about a new programming language but a sort of ruby language
preprocessor that can be called on-top BDD activities to enforce security
checks.

I just created an empty repository, I called [coat](https://github.com/thesp0nge/coat). 
This will be my summer project. More posts about it will follow.


<sup>1</sup>: in the 2000, the "real" world in the Italian IT was asking for
Java developers to build system integrations squadron of glue code builders. I
was young. Take pity on me.
