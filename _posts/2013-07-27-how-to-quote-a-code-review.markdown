---
layout: post
title: "How to quote a code review"
date: 2013-07-29 09:00
comments: true
published: true
featured: false
categories: appsec code-review sast static-analysis project-managers it-managers quote quotation 37signals myth forecast
thumb: planning.png
level:
hn: 
rd: 
---

A premise: I don't trust gantt and fancy IT project managers' document where
every project step fits in a perfect order without dealing with the
unpredictable.

Planning is guessing. The more information you have, the more close to reality
your guess will be. Without any hint, when you say you will take 5 days to get
the job done you are just shooting a number.

<!-- more -->

In their best seller [http://37signals.com/rework]("Rework"), guys from
37signals do have strong opinion about plans and project timing.

{% blockquote Rework, Jason Fried & David Heinemeier Hansson %}
Unless you’re a fortune-teller, long-term business planning is a fantasy. There
are just too many factors that are out of your hands: market conditions,
competitors, customers, the economy, etc. Writing a plan makes you feel in
control of things you can’t actually control.
{% endblockquote %}

Of course you need to have your project well organized with a sequence of
actions you have to deal with but you need also a lot of information to make
good planning. And most of times, nobody can tell you that much.

So let's figure it out a possible scorecard for a successful security code
review.

## Some basics

First of all, this scorecard and all the assumptions I'll make, they work for
me. Maybe you'll find them useful (and I'll happy about it), but maybe you want
to tune them in order to fit your needs.

The overall quote in terms of days relies on a key ratio that it's completely
up to the application security specialist. It depends his seniority and how
many code reviews he did. 

This values is the lines of code you can review on average (no matter how) in a
day.
Let's say you are an experienced appsec specialist. You do a lot of security
code reviews in a year, and you've got some supporting tool and a bunch of
heavly customized scripts you wrote during the years.
You know, because you calculated, that on average you can review X lines of
code in a working day. This is your base starting pint of your quotation.

Now, you apply some modifiers to your loc/day value.

## Do you assess software developers?

A critical point in a code review is that you're dealing with source code wroet
by people you're not familiar with. You must understand how they work, if they
have continous integrations, if they relies on a centralized versioning system,
if they do pair programming, if they have some security coding guidelines.

It's also very important to understand the story behind the source code you're
reviewing. Which is the source code goal? Which is the system architeture? How
many developers work on it? Which are the users interaction with the code? Is
there a user interface? Is it easy to use? Is it well designed? How the source
code is logically organized?

Assessing developers this way can give you an idea on the source code taken as
a black box.
Let's now assign score for this point.


<span class="question">
Before starting the code review, did you talk with developers assessing their
workout gathering some preliminary information?
</span>

<table class="table-striped table-bordered table">
  <thead>
    <tr>
      <th>Answer</th>
      <th>Modifier</th>
    </tr>
  </thead>

  <tbody>
    <tr><td>Yes</td><td>+2 points</td></tr>
    <tr><td>No</td><td>-2 points</td></tr>
    
  </tbody>
</table>

## Do you master the target technology?

Let's say it very clearly. If you call yourself a security code reviewer, you
**must** be able to write code. Reasons are very simple. You must write patch
proposals to the target source code, and you must understand the source code
you deal with.

If you're one of that guys, selling automagically generated source code reviews
report made by an automatic scanning tool and you called sourceself a security
specialist, you can stop reading this post and please forget this blog.

Remember, developers don't trust you. You must convince them you're familiar
with their gergo and with their work.

<span class="question">
Are you familiar with the technologies used by developers in the project you're
working on?
</span>
<table class="table-striped table-bordered table">
  <thead>
    <tr>
      <th>Answer</th>
      <th>Modifier</th>
    </tr>
  </thead>

  <tbody>
    <tr><td>Completely</td><td>+5 points</td></tr>
    <tr><td>Partially</td><td>+2 points</td></tr>
    <tr><td>I'm not that confident</td><td>-2 points</td></tr>
    <tr><td>I don't know that programming language</td><td>-5 points</td></tr>
  </tbody>
</table>

## Are you happy?

You may think you don't influence yourself the quote but you're completely
wrong. How good you can perform is connected on my aspects of your life. If you
feel good, you'll perform good; otherwise you may lack in concentration adding
some overhead in terms of day spent trying to solve a problem.

So, sit down and try to make a quick evaluation about your stamina level in the
days you'll perform the review (of course, the unpredictable will be there).

<span class="question">
How do you evaluate your quality of life? Are you happy? Are you peaceful and
well organized? Do you feel under pressure?
</span>
<table class="table-striped table-bordered table">
  <thead>
    <tr>
      <th>Answer</th>
      <th>Modifier</th>
    </tr>
  </thead>

  <tbody>
    <tr><td>I can manage any kind of workload. I'm pretty organized and most of times stress is under control</td><td>+5 points</td></tr>
    <tr><td>I'm kind a of organized. I'm quite but sometimes I miss some milestone due to high amount of work</td><td>+2 points</td></tr>
    <tr><td>I'll do a lot of freelancing those days / I've got a lot of family stuff to do</td><td>-2 points</td></tr>
    <tr><td>It will be a mess, I'll experience a very hard workload but I can do it... I hope</td><td>-5 points</td></tr>
    
  </tbody>
</table>

## Now make some math

Now that you tried answering scorecard questions, it's time to sum up your
score.

The following table will give you an idea on how to add something to your code
review quote.

First, you know approx how much big is the source code you're going to review,
let's call this number Y lines of code. The X lines of code versus days is your
basic "code review speed".

The time you will spent (theoretically) on this code review is: Y/X days.

<table class="table-striped table-bordered table">
  <thead>
    <tr>
      <th>Score</th>
      <th>Amount to add to your quote</th>
      <th>Note</th>
    </tr>
  </thead>

  <tbody>
    <tr><td>Q &gt;= 10</td><td>+5%</td><td>Believe me, you missed something in your anwswers. It's alwasy better to include a soft area just in case something unpredictable will happen.</td></tr>
    <tr><td>7 &gt;= Q &lt; 10 </td><td>+10%</td><td></td></tr>
    <tr><td>5 &gt;= Q &lt; 7</td><td>+20%</td><td></td></tr>
    <tr><td>2 &gt;= Q &lt; 5</td><td>+20%</td><td></td></tr>
    <tr><td>-2 &gt;= Q &lt; 2</td><td>+30%</td><td>At this point I strongly encourage you rethinking something in preliminary steps when you create a quote</td></tr>
    <tr><td>-7 &gt;= Q &lt; -2</td><td>+50%</td><td></td></tr>
    <tr><td>Q &lt; -7</td><td>+75%</td><td>Critical here. You don't have enough information to make a good quote, you're completely blind man.</td></tr>
  </tbody>
</table>

## Off by one

I do really think planning is voodoo. You can't always estimate in advance how
many days you'll spent in a code review. I use this scoreboard myself to drive
decisions and give a number and 80% of times I make a good estimate. The other
20% however it turns the activity in a completely mess, most of times because
project managers don't give too much information about the code or developers
give poor answer to preliminary questionnaire.

The idea is feel comfortable with your skill and with your swiss army knife
tools and methodologies. Good estimates will follow... however I do think it's
better to make good code reviews, of course.

Enjoy it!
