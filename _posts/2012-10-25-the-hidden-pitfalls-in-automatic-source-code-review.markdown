---
layout: post
title: "The hidden pitfalls in automatic source code review"
date: 2012-10-28 09:02
comments: true
published: true
featured: false
tags: appsec code-review sast commercial-tools mindmap threat-modeling safe-coding owasp owasp-esapi awareness
thumb: pitfall.png
hn: http://news.ycombinator.com/item?id=4712039
rd: http://redd.it/129mcp
---

_Disclaimer: this is an in depth post about pitfalls in security code reviews.
A codesake.com focused post is available on [codesake.com blog](http://blog.codesake.com/the-pitfall-of-automatic-code-review-what-codesake-won-t-become.html)_

Introducing application security in an IT organization is really tough. Your
goal is to integrate security by steps in software development processes and
infrastructure server management. You **must be a good diplomat** and you have
to take care **not to slow down too much** other people work.

Sometimes newly born application security teams start with a static analysis
tool. With the report in their hands, they can leverage some software
vulnerabilities. However they can either fall in one of the following pitfalls.

** UPDATE **

_30.10.2012: Some people reported that this post has quality content but it's
written with a very poor quality English. I double check words spelling and
grammar. I hope it's more readable now._

<!-- more -->

## Pitfall 0x01: your tool is enough

There are a lot of widespread misconceptions about source code static analysis.
The scariest one is that having a SAST tool, no matter how good it can be, is
the goal you have to reach.

Application security is a process that engages your developers, project managers,
software architect and even marketing people.
It's quite common for big companies to [outsource non business critical websites to web agencies](http://armoredcode.com/blog/are-web-agencies-the-new-security-threats-in-2013/) and neither you nor web agency guys are skilled enough to think about software security.

Let both non technical people and non security people aware about your risk is
the master key to the success.

Your application security team must start an awareness program and introduce IT
people to the risks your core business might occur if attacked.

You have to _show_ how a web application attack is made.
Take [OWASP WebGoat](http://code.google.com/p/webgoat/) as target and break it.
It could be even better if you'd use an internal version of your web
application as example.

The goal here is not to tell developers that they have _poorly written code_,
but to train them about risk and show what an attacker can do with their code.

A very useful tool it would be a sort of coding dojo for developers to quick
mockup the code. They will have a quick feedback about code security level and
project manager can have some KPI indicators about their team improvementes.

## Pitfall 0x02: early stages aren't important enough

I was engaged in a number of late running code reviews. The code was already in
production and no one had clues about application security and web application
perimeter was very foggy.

_How can I access this page? Which protocols do you support? Is there any API?
Is your database well protected?_

For development team that doesn't think application security as a key component
of their workflow, those questions are either not important at all.

However a quick and dirty threat model can address mitigations task you can suggest
when you find something. You must give developers some added value.

Engage the design team with some friendly discussion driven threat modeling
question.

This table can be used as quick start reference:

<table class="table-striped table-bordered table">
  <thead>
    <tr>
      <th>Question</th>
      <th>Answer</th>
    </tr>
  </thead>
  <tbody>
    <tr> <td>Is the application published internally?</td><td>&nbsp;</td></tr>
    <tr> <td>Does your application have an administrative backend?</td><td>&nbsp;</td></tr>
    <tr> <td>Are your sensitive information protected with HTTPS?</td><td>&nbsp;</td></tr>
    <tr> <td>Are you going to expose some APIs?</td><td>&nbsp;</td></tr>
    <tr> <td>How is it segregated your environment? Is there any firewall or WAF?</td><td>&nbsp;</td></tr>
    <tr> <td>Where is the database? Is it served in a separated server?</td><td>&nbsp;</td></tr>
  </tbody>
</table>

## Pitfall 0x03: a tool's output is something I need

Yesterday I launched a commercial static analysis tool over
[latest wordpress version](http://www.wordpress.org), 3.4.2 at the time I'm
writing this post.

> As a security specialist I think that commercial SAST tool vendors have a great
> marketing team instead of a powerful knowledge base.

I expected to find no more than a couple of things; wordpress 3.4 is mature and
strong enough and its team just released a lot of security fixes.
problems.

Instead, the tool reported more than 150 high level vulnerabilities. There are
some suspected SQL Injections, OS command injections and tons of Cross Site
Scriptings.

_That's insane._

If I had to manually review all high level severity findings, it will take me days.
It would be clever to manually review some interesting code snippets related
about DB post saving or HTTP requests handling.

The second thing that drove me crazy was the following. How can be a just
released security patch vulnerable to more than 150 vulberabilities? Am I lucky
enough and I found 150 possible 0-days for wordpress? In this case I will make
a lot of security advisories increasing my popularity.

Instead, having found 150 false positives is a more realistic conclusion.

A tool has not to promise 100% false positive free deliverables. A lot of
vendors make this claim and, as a security specialist, I think their marketing
team skills are agrater then their tool knowledge base.

Running a mix of hybrid analysis and fuzzing over suspected sanitizing routine
will lead me to provide better results.

I mean: if a tool detects that a tampered variable can be used in the
presentation layer after a routine called _make-your-string-safe_ it can't say
for sure anything about this attack path. It must make some fuzzy attack over
_make-your-string-safe_ in order to understand if it's a validation routine and
if it's performing well.

## Pitfall 0x04: a tool's output is something THEY need

Source code static analysis tool output is not made for developers.
Application security specialists write security tools and they think all people
in the IT world can understand that fixing a XSS is just a matter of _escaping your input_.

It doesn't work this way. I'm pretty sure that if you take, let's say, 5
developers you will find 5 different algorithms for escaping the input.

Developers are either focused on totally different aspects of a web
application. They are thinking about SEO optimization, about layout, about
business logic, about bugs. He doesn't know anything about what do you really
want him to escape since he doesn't know all the possible ways you can write a
XSS attack vector. And this is completely fine.

Even if the report it has been written by an application security specialist,
the target audience are developers. Code snippet and programming design patterns have to
be used instead of theoretical security considerations (they still great for
remediation introduction, just for awareness sake).

> Testing only the static analysis bit (or just making a penetration test) it's
> like testing car brakes without checking if the engine turns on.

Output for security tests won't be in term of _fix your code for injection_ or
_if you're a java application then use this ESAPI class_ (if testing a .NET or
Ruby project).

Fixes has to be as specific as possible and they must be written in the same programming
language used by the application.
Fill your report with third parties libraries suggestions, like
[OWASP ESAPI](https://www.owasp.org/index.php/Category:OWASP_Enterprise_Security_API),
and strongly encourage your development team to use opensource code that
eventually it can be reviewed for security issues instead of closed third party
libraries.

This security recommendation, can be an example for an application written in
ruby programming language:

* put owasp-esapi-ruby gem into your Gemfile

```
...
gem "owasp-esapi-ruby"
...
```

* make a bundle install

```
$ bundle install
```

* include Esapi::Sanitize module in your class

```
...
include Esapi::Sanitize
...
```

## To reduce false positive emissions, you must go hybrid

As said earlier in this post, static analysis even when using the most powerful
commercial tool out there, doesn't cover all the possible security check you
can do over your code.

If you don't actively stress some piece of code with tampered data you can't
say if it's a good sanitization routine or if it's a simply vulnerable piece of
code.

Testing only the static analysis bits (or just making a penetration test) it's
like testing car brakes without checking if the engine turns on.

The following picture is a mindmap describing an hypothetical ecommerce
checkout workflow.

![]({{site.url}}/images/simple-mindmap-one.png)

Let's see what both static and analysis tools will say about this.

### The static analysis

_Mmh the code here has poor quality. KPIs indicators say that cyclomatic
complexity is too high and there are not a lot of comments around this piece of
code. It may become an unmaintainable snippet if no documentation will be added
as soon as possible.
Oh, look... there is a possible SQL injection here, users can control variable
cartID from the HTTP requests and it is used directly in a SELECT out there in
payment_checkout() method.
Is it also possible to make a Cross Site Scripting in the payment confirmation
screen._

### The penetration test

_Here is HackerScan commercial tool. Your digital certificate doesn't match the
hostname, just move into the trash and spend some bucks to a valid one. It
seems also your website is vulnerable to Cross Site Request Forgery; don't be
afraid, almost 90% of websites around there have the same problem.
Look, if I tamper the cartID with a letter instead a number I'll cause an
unmanaged exception, can you please tell your developers about try and catch?_

### Go hybrid now and start fuzzing

Looks weird? You have to image the two tests like the two faces of the same coin.
What about the Cross site scripting or the SQL Injection the static analysis is
pretty sure it will happen but the penetration test didn't report?

In my experience, I'll double check with another dynamic testing tool, if two
tools weren't able to exploit a suspected vulnerability then I make a further
manual check and if also the manual check will fail, I'd mark this as false
positive.

You may ask at this point, how can a tool that looks at the source code detect
a false positive?

Well the tool itself can't say nothing about the controls() routine that is
here to make input sanitize.
A clever approach is would be, save the controls() routine as a standalone
piece of code and start stressing it with attack patterns looking how it
behaves.

If code won't affect in any way the attack patterns leaving the data content
unmanaged then the data flow can be tainted, otherwise you have to mark this
attack path as sane and you have to discard both the SQL injection and the
cross site scripting you previously detected.

In order to have this kind of approach running, your tool must be able to
either compile or launch the code you want to fuzz creating on the fly a
testing source code like:

``` ruby

patterns = load_attack_patterns # load our knowledge base content

patterns.each do |pattern|
  output = controls(pattern)

  tainted = output.tainted?
end

```

Your home-brewed fuzzing framework must provide:

* a knowledge base of attack patterns
* a tainted? method that it has to be used if the the output is tainted or not
* some parsing facilities to understand parameters to be submitted to the
  suspected tainted method


## Off by one

Application security is not buying the latest commercial tool and start telling
developers they must patch their code.

It starts from a security awerness program about safe coding basics and it
follows introducing threat modeling with your software architects in your web
application early stages.

Application security is giving your developers safe coding guidelines with real world coding example, not only theory.

Then you can start using hybrid static analysis combining tools
and your expertise to provide developers real value preparing a reports thay
can understand.
Use source code instead of pre formatted IT security remediation text.

Use opensource tools you can customize to fit best your needs instead of
commercial tools. Opensource alternatives perform as good as their [commercial counterpart](http://sectooladdict.blogspot.it/)

Enjoy it!

_Pitfall street in Leeds picture is credit by [Tim](http://www.flickr.com/photos/atoach/)_
