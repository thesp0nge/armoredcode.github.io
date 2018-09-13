---
layout: post
title: "Let the sake for code to flow"
date: 2013-11-18 18:37
comments: true
published: true
featured: false
categories: codesake codesake-dawn dawnscanner code-review sast ruby
thumb: codesake-first-scan-title.png
level:
hn: 
rd: 
---

**UPDATE**
For a mistake this post appeared today on
[armoredcode.com](http://armoredcode.com) without the text. Reason is that I
created a placeholder to remember me to work on this.

I'll fill with the text now. Sorry for this.

<!-- more -->

## A blasting engine

Few days ago I released the [version 0.79.99 of codesake-dawn](http://rubygems.org/gems/codesake-dawn) 
static source code analyzer for application written in ruby programming language.

I decided not to call it _0.80_ because of the project roadmap and because some
major feastures need to be introduced. I decided however to release it because
it was the first version making the [application security saas portal I'm working on](http://codesake.com) to work.

![]({{site.url}}/images/codesake-first-scan-title.png)

Taken from the project roadmap, since the latest version I introduced 14 new
security checks, a combo security check designed to allow flexibility in tests
where more conditions has to be met and I introduced the ability to just scan
Gemfile.lock for inherited security issues. 


``` 
Version 0.79.99 - codename:oddity (2013-11-14)

* adding test for CVE-2013-2065
* adding test for CVE-2013-4389
* adding test for CVE-2010-1330
* adding test for CVE-2011-0446 
* adding test for CVE-2011-0995
* adding test for CVE-2011-2929
* adding test for CVE-2011-4815
* adding test for CVE-2012-3424
* adding test for CVE-2012-5380
* adding test for CVE-2012-4522
* adding test for RoRCheatSheet\_1
* adding test for RoRCheatSheet\_4
* adding test for RoRCheatSheet\_7
* adding test for RoRCheatSheet\_8
* Fix issue #1. You can read more about it in TODO.md
* Added API to scan a single Gemfile.lock using -G flag

```

The latest is the one making [codesake.com](http://codesake.com) to work.

## An application security SaaS

There are lot of great application security saas or startups
([codeclimate.com](http://www.codeclimate.com/), [klin from fogcreek
folks](http://www.fogcreek.com/kiln/)),
[brakeman.org](http://brakemanscanner.org/), and others). Codesake.com is my
answer to the need of application security in ruby powered world.

The goal is to provide both static than some dynamic security tests with
codesake's family gems (that are completely opensource and they always will
be) and provide RESTful APIs people can use to ask for security services.

I won't charge you like big companies. I promise.

This post is important to me since I worked on #dawnscanner since April when I
officially started the [sast engine](http://www.gartner.com/it-glossary/static-application-security-testing-sast)
as a separate part from the dynamic [web application penetration testing gem](https://github.com/codesake/codesake-dusk).

Before April, in codesake.com codebase there was a library called codesake that
eventually it would turned as a standalone gem, but I wasn't that sure about
the licensing model.

After the amazing experience in [Railsberry](http://railsberry.com] I decided
to split off the two engines as two big standalone opensource projects.
codesake-dusk is built using tons of small piece of ruby code I wrote in
realworld [activities I wrote in this blog](http://armoredcode.com/blog/categories/pentest-with-ruby/).

codesake-dawn is growing faster as a project, with just a couple of slowdown
later this year.

In the future I'll try to built a successful and profitable SaaS around those
two rubygems, enjoying that much that application security is becoming
mainstream in ruby community.
