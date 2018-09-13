---
layout: post
title: "Which is the most secure programming language ever?"
date: 2012-07-11 08:34
comments: true
published: true
featured: false
categories: builders appsec awareness programming-language 
hn: 
rd: http://redd.it/wdk1a
---

Sometimes I was asked about which is the most secure programming language to
use in real web applications.

The short answer is: **no one** or, better, every programming language is
secure enough to be used in the real world if used with security in mind.

<!-- more -->

Lazy project managers or not-so-skilled developers are used to look for
shortcuts to achieve secure software. Sometimes building a security product it
is seen as the _panacea_ solving all the issues that can arise.

This is not true. Security in computer science is a process rather than a
product to buy. 

You must keep in place a [vulnerability management policy](http://armoredcode.com/blog/even-before-your-secure-coding-dot-dot-dot-patch-your-server/)
in order to keep your servers constantly up-to-date. 
You must have your [software architecture](http://armoredcode.com/blog/understanding-your-attack-exposure/)
documented and built with security in mind.
You must not rely on your users but [enforce security constraint](http://armoredcode.com/blog/leakedin-and-the-salt-and-pepper-sauce/)
in your code to drive them out of risky paths.

There is no firewall you can buy that makes your customer safe for a poorly
written algorithm or from an unsafe cryptographic function you choose to use.

And it is completely programming language agnostic topic.

You can write secure code in Pascal, C, Ruby, Perl and even Visual Basic.
[TIOBE index](http://www.tiobe.com/index.php/content/paperinfo/tpci/index.html)
must be taken as popularity index among developers not as report about the
language you must use to write secure or robust software.

Remember that you must be comfortable with the language you will use. If you're
an experienced Delphi Senior developer, it is likely you will write good
quality software in Delphi rather than Ruby or a more popular one.

So choose the technologies you are more familiar with. Go in deep with them, be
curious and use some test driven development practice in order to test the code
before deploying.

There is no magic bullet for software security.

## The role of web frameworks

Of course my thoughts about programming language are overriden by frameworks published to help developers in writing web applications.

[Rails](http://rubyonrails.org) introduces an anti-[cross site request forgery](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF\))
token and some goodies for HTML escaping making difficult a [Cross Site Scripting](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS\)) to be
exploited successfully.

[.NET](http://www.microsoft.com/net) introduces the VIEWSTATE that makes
parameters tampering impossible by server side validation performed by the
underlying application layers. Microsoft either released an [anti XSS library](http://msdn.microsoft.com/en-us/security/aa973814.aspx) available to
everybody and that it can be used in .NET projects.

[Spring](http://static.springsource.org/spring-security/site/features.html) introduces security feature that they can be embedded in J2EE projects.

I'm pretty sure that other frameworks introduce similiar features and if they
don't, there is the [OWASP Esapi Project](https://www.owasp.org/index.php/Category:OWASP_Enterprise_Security_API)
you can use to enhance your web application security level.

