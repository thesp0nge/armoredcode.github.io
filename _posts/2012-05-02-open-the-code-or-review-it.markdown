---
layout: post
title: "Open the code or review it: Oracle CVE-2012-1675"
date: 2012-05-02 08:09
comments: true
published: true
featured: false
tags: builders code-review database dbms oracle cve-2012-1675 cve tnslistener man-in-the-middle
hn: http://news.ycombinator.com/item?id=3917954
rd: http://redd.it/t2z9f
---

## I'm fine with Oracle, but...

This post start with the latest vulnerability
[CVE-2012-1675](http://urlin.it/2f2c1) about Oracle TNSListner.

A [Man in the Middle](http://urlin.it/2f2c7) attack is possible against the
Oracle TNSListner to hijack regular users connections. The advisory says also
that it is possible to make denial of services and full compromise the remote
database integrity.

The attacker **does not** need a pair of valid credentials, it has just to
reach the listener via network.

A funny way to start the week.

<!-- more -->

## It all started 4 years ago

Koret, the security researcher that posted on full disclosure, first reported
the issue to Oracle [4 years ago](http://urlin.it/2f2cf). It thought the vuln
was fixed so long ago, but it doesn't.

Oracle admitted that it was not able to provide a patch since those will ruin
all regression tests for older DBMS versions that are suffering for the
listener poisoning attack.

4 years ago my son yet didn't exists, nor I was married and I was wearing
Taekwondo ITF green belt.

Since 4 years your TNS Listener are exposed to a remote and critical
vulnerability, that can't be fixed in a while and the vendor was aware of it.

4 Years in the IT world is like a decade, but in the Information Security 4
years last like a century. Hopefully no TNSListners are exposed to the
Internet, however consider how much database lays unprotected in flat corporate
networks.

> Since 4 years your TNS Listener are exposed to a remote and critical
> vulnerability, that can't be fixed in a while and the vendor was aware of it.

## The quest for being certificate

You have also to consider that a lot of third-party software were build in the
meantime over such vulnerable versions and were certifified to run only on
certain Oracle versions.

All of those are exposed to same vulnerability and it's **not sure** will
certify against the new Oracle Critical Patch that will be released to fix (we
all hope this time for real) the vulnerability).


> A lot of third-party software were build in the meantime over such vulnerable
> versions and were certifified to run only on certain Oracle versions. All of
> those are exposed to same vulnerability and it's **not sure** will certify
> against the new Oracle Critical Patch that will be released to fix (we all hope
> this time for real) the vulnerability).

It happens all day, some Oracle Critical patches can't be applied because the
**third party vendor doesn't certify its product to be compliant with that
one**.

Big vendors are just lazy and they certify compliancy with a patch usually with
some months delay. But what about mid-size companies or even custom made
products?

Certifications make _stakeholders_ so comfortable about nothing bad will happen
to their company.

After all, the software they paid for is certified doesn't it?

## Closed versus Open. Being certificate or just being smart?

This is not a rant about _closedsource_ versus _opensource_. None in between of
them carries a silver bullet for vulnerabilities or software errors.

Of course, _opensource_ software can be _theorically_ reviewed by a lot of
smart people... but this doesn't happen in the real world. People, even in the
_opensource_ world needs to deliver faster and smart solutions. The time to
investigate a software for security issues is lacking and more important...
resources lack even more.

A true fact is that when a security issue is reported in mainstream
_opensource_ project, the reaction time is misurable in **days** _(remember: 4
years)_ and all is exposed and disclosed.

More important is that when a framework or a technology is released, it's not
certified to run on certain operating system version, or only if the DBMS has
not installed a patch, or with certain versions of java.

> A true fact is that when a security issue is reported in mainstream
> _opensource_ project, the reaction time is misurable in **days** _(remember: 4
> years)_ and all is exposed and disclosed.

Certifications are the most hilarious bit of all the story.

You tie your product to another product version make it immutable to changes in
time. No one will ever upgrade operating system or other daemons because this
would be break the certification.
After each vulnerability assessment or after each new critical patch the vendor
will publish, no one can upgrade because this will break the certification.

This is ridiculous. It's like building the new [facebook](http://facebook.com)
saying that it will run fine but you don't have to use _acer_ (put here the
hardware manufacturer you like most) and you don't have to access the
application on Mondays.

## Some rules software developers must learn from this story

1. You write software on top on basic layers such as an operating system, a
   dbms or an application manager. You must people leave freedom of choice.
   Don't tie your code on version x.y.z. It's a risky behaviour and it tells
   the world your code is not rock solid but it inherits stability from the
   underlaying layers.
2. If you really need to lock your application to a certain vendor release,
   please **follow the vendor security patches** and upgrade your certification
   agreement as soon as the vendor released the patch. It's important for security
   teams to tell people to apply patches to mitigate a breakin exposure.
3. Thinks in term of layers when you write your code. Most of the code can be
   dbms or operating system agnostic, the code it can't be like that make it
   sure it will be as far as standard as possible so to be easly ported in
   different dbms-es or application servers or operating systems.
4. Security is critical, for your customers and for your stakeholders.
   Vulnerabilities are discovered every day and software must be kept up to
   date to avoid security incidents. You are part of a security process, you
   really don't want to stop.

## An important rule for big software vendors

Independent researchers make a great work to you in sending advisories, or in
finding vulnerabilities. May be they are smarter than your security team.
Please read their work and close the breaches as soon as possible.

4 years is not an acceptable timeframe, moreover if the reason is that some
regression test would fail.

## On the other side.

The 0day exploit in this video.

{% include youtube.html id="hE3-AkxSX3w" %}
