---
layout: post
title: "I don't care if app is unsecure, it's friday I'm in love"
date: 2013-04-19 07:59
comments: true
published: true
featured: false
tags: appsec survey wapt code-review project-manager boss stakeholder
thumb: friday-im-love.png
level:
hn: https://news.ycombinator.com/item?id=5575418
rd: 
---

A month ago I opened a "one question only" survey on
[surveytmonkey](http://surveymonkey.com). 

I asked _"Why you don't make any web application penetration test when I deploy
a new web application (or a new feature)?"_

I collected 41 answers after advertise the poll on
[linkedin](http://it.linkedin.com/thesp0nge),
[facebook](https://www.facebook.com/armoredcode) and on
[twitter](https://twitter.com/armoredcode).

I asked also the [Italian Ruby mailinglist](http://lists.ruby-it.org/mailman/listinfo/ml) 
that is full of great ruby specialist, startuppers and makers.

Let's analyse the results

<!-- more -->

## Slightly intended to turn on provocation
You noticed right, I'm a **provoker**. I eventually could asked _Do you test
your application for security issues before deploy it?_ to let people say
easily _Yes we do make a lot of tests_ but in my experience (I'll be always be
happy in being contradicted) the percentage of people applying security tests
to web code is **poor**.

Sorry to be so dramatic, but it's quite true that most of people in small and
medium business don't care about security (or test overall).  

People in large business... well, they don't care too but this poll wasn't
answered by those kind of guys.

If all people eventually make security test over their code, this blog won't be
useful anymore isn't it?!?

## Results

<table class="table-striped table-bordered table">
  <thead>
    <tr>
      <td>Answer</td>
      <td>Votes</td>
      <td>Percentage</td>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td>You're wrong. I do make a web application penetration test when I deploy a new web application or a new feature</td>
      <td>8</td>
      <td>23,5%</td>
    </tr>

    <tr>
      <td>No budget. Security costs are too high for us, we're a startup and we're focused on business first</td>
      <td>16</td>
      <td>47,1%</td>
    </tr>
    <tr>
      <td>No need to. We're a big development team. Our code is robust and strong. We won't occur in any security incident. Ever</td>
      <td>1</td>
      <td>2,9%</td>
    </tr>
    <tr>
      <td>No time. We are missing our deadlines. We don't have time to spent in security tests. We are safe from risks. We have firewalls.</td>
      <td>4</td>
      <td>11,8%</td>
    </tr>
    <tr>
      <td>I don't care. Seriously, security is a word spent by sales men to
          sell antivirus or similiar stuff. I don't think my web application will be
          attacked by "so called" hackers.
      </td>
      <td>5</td>
      <td>14,7%</td>
    </tr>
  </tbody>
</table>

![]({{site.url}}/images/do_you_test_for_security_poll_results.png)

## Other answers 

On the poll there was also an open answer box where people can leave their own
answer if non of the above fitted.

{% blockquote %}
No the application is deployed on Windows Server which is already secure
{% endblockquote %}

{% blockquote %}
Our managers don't care about that... sigh.
{% endblockquote %}

{% blockquote %}
I approach security from the development side (static analysis, code reviews
etc) and don't expect later pentests lead by the same dev team to improve
security, but I do run automated tools which have proved useless over time.
{% endblockquote %}

{% blockquote %}
I don't have enough time and money to invest in these. Is it possible to automate them?
{% endblockquote %}

{% blockquote %}
It's a mix of "No time."/"No Budget" and another one you've not specified: "No
knowledge" :-) Usually, we don't have the necessary knowledge to perform an
efficient pen test session and in order to obtain that knowledge we should
invest a too high amount of time. I know it's a vicious circle that in the long
run doesn't pay very well :-)
{% endblockquote %}

{% blockquote %}
No time. We are missing our deadlines. We don't have time to spent in security
tests. We know what pentest is but we consciously decide to skip it. And pray
that no skilled hacker will ever turn his eyes to us.
{% endblockquote %}

{% blockquote %}
I wouldn't know how to perform penetration tests. But I would like to know more about them.
{% endblockquote %}

## My comments

Looking at the poll results I can see a good number of people (24%) that they
run application security tests (their own or asking a freelance to do that).
So, as average 2 out of 10 web applications out there are tested for security.

Other 8 out of 10 are not tested for security issues and the main reason is
that people have **no budget**.
But, how does it cost a good web application penetration test? And how much is
it compared to the hidden costs of rewriting the app from scratch or monkey
patching it after a SQL injection?

Even more, how much is it compared on your brand damage and all che costs
related to a data loss after a security break-in? If you would experience a
security issues, your competitors can take a competitive gain over you. You
will potentially lose customers. Are you sure that you can **really** efford
this risk?

It's like designing a brand new car. You pay designers and engineers to create
a super car with great design and outstanding performances. You car is great
and intended to choosy customers who want to pay large amount of money for a
good service.
**But** when you design the car you don't have enough budget to implement a
full ABS plus stability control system, so you will not implement a top
solution and your car fails on the market.

Application security is your breaking system. You must take care of it if you
want to build a top class product. If you don't, may be is a good product until
someone (for not a predictable reason) will break into it, steal your customers
data and make your business to fail.

For people who don't care, well maybe they are even not reading this blog or
they don't care about IT security at all. I discourage people from ignore the
IT security issue... in case of break-in your business or your online presence
can be seriously compromised.

Open answers open two different points:

* I'm not skilled enough / I don't have enough time to do **also** application
  security. Good, and that's why there are application security specialists you
  can engage to help you in security tests. For the costs issues, ask a quote
  before and then evaluate if the money you will save can deal against the money
  you will lose if successfully attacked.
* Automated penetration tests. For sure you can. There are commercial tools out
  there and there will be [codesake.com](http://codesake.com) soon to ask for
  application security tests. I **strongly** encourage you also to do some
  manual tests since a tool can make a 100% coverage of your application no
  matter how good it is. It's a clever idea to have an application security
  specialist to integrate tools with some manual check.

And you? What do you think about this topic? Which are your experience?

Do you make web application penetration test when you deploy a new web application or a new functionality?

If not, why you don't introduce application security in your daily workflow? Tell me yours.

