---
layout: post
title: "Some security tips for ruby hackers: prelude"
date: 2012-06-12 08:05
comments: true
published: true
featured: false
tags: builders breakers ruby webapp gems pentest speech rubyday owasp owasp-testing-guide pentest-with-ruby
hn: 
rd: 
thumb: hacker-for-dummies.jpg
---

Next Friday I'll give a [talk](http://rubyday.it/talks/2/) about using ruby and
gems to quick test a webapp for security issues.

It won't be a "Evil attacker for dummies" training nor a "Appsec Certification"
talk. 

I will simply show some basic security checks to make before deploying an app
and how to use some opensource ruby gem to help you.

This week I'll blog about the roadmap in building my talk
 
<!-- more -->

## Prelude

### Why Ruby?

The specific programming language you choose to make some security checks it's
not important. You can make the all the tests either in Java, C, C#, Bash
scripting or with a Turing machine. 

I choose [ruby](http://ruby-lang.org) mainly because:

* it's sexy to be used in writing web apps with great frameworks like
  [rails](http://rubyonrails.org) or [sinatra](http://www.sinatrarb.org);
* it has a well documented [api](http://rubydoc.info) and developers are
  trained to write documentation;
* Net::HTTP and mechanize are amazing pieces of code

### Why testing?

If you just forgot [linkedin password leakage](http://armoredcode.com/blog/leakedin-and-the-salt-and-pepper-sauce/) I
can recall you that there are a lot of criminal crews out there attacking
websites to steal accounts, infecting people with malware and taking servers
control. 

Testing for security issues is important but today I don't want to bother you
with recommendations about [being aware](http://armoredcode.com/blog/understanding-your-attack-exposure/) of the
problem.

### What to test?

The [Owasp testing guide](https://www.owasp.org/index.php/OWASP_Testing_Guide_v3_Table_of_Contents)
will be the framework I'll use underneath my talk.

The Testing Guide by [owasp](https://www.owasp.org) is one of the greatest
piece of documentation Owasp community has ever wrote. It's a 300-or-something
pages volume describing the most important checks a secure web application must
pass.

![]({{site.url}}/images/kid-with-suit-testing.jpg)

Here there is a list of security areas we will cover and we will find a ruby piece of code to manage:

1. Information Gathering
2. Configuration Management Testing
3. Authentication Testing
4. Session Management
5. Authorization Testing
6. Business logic testing
7. Data Validation Testing
8. Denial of Service Testing
9. Web Services Testing
10. AJAX Testing

{% blockquote %}
Owasp testing guide driven assessments are a walk-through between those top 10
topics that they can recall the Owasp Top 10 security risks
{% endblockquote %}

Covering all of those points it will take more than days in a serious in deep
talk. I'll just give an overview on Friday but I'll go in deep here in my blog.

## Information gathering

I guess what are you thinking: "_it's my web application. It's not even
published on the Net. I have all the information about it, what re you talking
about?_"

You must start thinking as one of your _evil_ users. When you put online
something, you almost lose control over it. Your users become the real
protagonists, they registers to the site and they take the stage asking for
services and obtaining results. **But** we are not in an ideal world and users
can be either lawful people asking for services or unlawful people who wants to
crack the site.

And the first step the latters have to solve is to gather more information they
can to make targeted attacks.

That's why it is truly important you start thinking about how much information
you will disclose for free to a potential attacker, just deploying your code.

## Configuration management

It's all about configuration. A well written web application can be compromised
if the administration backend is not password protected or there is no control
over API calls so a non authenticated user can delete items in the database.

What if a user can find a backup file containing passwords? What if source code
can be disclosed under some circumstances?

What if the password for _admin_ user is either _password_ or _admin_?

## Authentication, sessions and authorization

Writing an ad-hoc authentication, authorization and session management
framework can be risky if your application is designed to hit the market and
become a profitable startup.

Something an attacker will try is to SQL injecting something into your login
form to bypass the authentication query. It's the classic ' OR '1'='1 lame
attempt. But most of times it works and you're logged as _admin_ without
bruteforcing passwords.

Make sure your legitimate users can't call APIs or URLs they are not supposed
to even know because, in example, they don't pay for that service. 

Consider this scenario, you're allowing photo resizing API only to premium
service subscribed users. However you build the API link for photo resizing in
a nice to see web-2.0-javascript. Users can try to call that URL asking for
photo resizing and it will be up to the authorization schema to deny the
request. 

In the Owasp Top 10 slang, this is called _failure to restrict url access_, but
in terms of your startup it's called people that ask for a service without
paying for it... or better, without paying **you** for it.

{% blockquote %}
Telling stories about anonymous or other crews attacking systems for activism
can make your blog to have tons of readers.

Telling stories about a normal person can leverage your application weakness to
use your paid service free as in beer it's more efficient to make you
understand application security _do_ is a concern.
{% endblockquote %}

## Business logic 

Testing for business logic is the trickiest part of all the game. This is about
testing the logical flow you imagine when you wrote the code can't be
subverted.

Consider an online shop checkout system. What if we can checkout the goods
without passing to the credit cards details? What if we can inject an arbitrary
number as checkout result?

Yes, it's all about money and please believe me... ordinary people will exploit
a checkout weakness to purchase a mobile phone or an ebook for free if they
can.

You write the rules so make sure to write a strong business logic.

_This is an important check in a real world penetration test but it can't be
fully automated. The attacker must study and understand the logic behind the
application and then exploit it. A tool most of times can't do it very well_

## Data Validation 

Do you remember _Cross site scripting_ and _SQL injection_ attacks? Ok, it's
time to check them.

As you may wonder, making a full penetration test involved **much more** work
that just checking for a XSS or a SQL injection.

The good news is that submitting attack patterns and see how the code reacts
can be full automated here.

## Something not covered in this talk

If I'm lucky enough, I would spent all of my talk timeslot just to explain
people all the stuff they have to check with some example. For such a reason I
won't cover denial of service, web service or ajax testing.


## Wrap up

This is my talk backend. I would then try to give some ruby code (most of them
written by myself) to automated some tests.

The idea behind my talk is not to make my audience experienced in web
application penetration tests. I just want to make them aware that they can
introduce some tests in their daily programming workout.

And writing this post can help me in preparing the talk as well :-)
