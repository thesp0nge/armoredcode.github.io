---
layout: post
title: "Papa don't breach"
date: 2012-04-10 09:26
comments: true
published: true
categories: breakers breach vulnerabilities world summary
hn: http://news.ycombinator.com/item?id=3821304
---

Latest days, while recovering from Eastern's BBQ galores, I was hanging around
[my](https://twitter.com/thesp0nge) tweeter feeds and the most occurrent topic
was... security breaches.

<!-- more -->
![](http://www.cuttothechaseproductionsllc.com/Images/Affiliates/HullBreach.png)

Despite [this](http://armoredcode.com) application security blog is gaining
some traction and hopefully it will last with some readers in the next years,
tehre a lot of venerable security professionals talking about application
security out there.

But are developers out there aware of this? Do they care?

## UTAH Medicaid data breach caused by ‘a mistake’

A week ago, as reported on [this online article](http://www.sltrib.com/sltrib/news/53862397-78/security-medicaid-utah-health.html.csp)
for the Salt Lake tribute, a security breach occured in the [Utah department of health](http://health.utah.gov/) and more then
[800,000](http://www.sltrib.com/sltrib/news/53879423-78/breach-information-health-medicaid.html.csp)
people medical records were leaked from database by attackers.

From the Salt lake tribune article:

> Boyd Webb, chief information security officer for the Utah Department of
> Technology Services, said he was not ready to talk about who launched the
> server without setting the proper layer of security because the investigation
> of the incident is still active.

This is scary. The CISO is saying that somebody put online a server and the
security team **did not** check it for security issues. A security team
**must** be engaged in an online activity for both pre-release and post-release
security checks.

Of course all the media attention is on the eastern european bad guy that make
the breakin but my concern is also... how things like this can happen on a very
sensitive website? 

We are not talking about credit card numbers here. They are sure important but
the cards can be revoked as a countermeasure act. We are talking about
sensitive medical data, we are talking about people. Can we say the only one
bad guy here is the one who breaks or also people that didn't take all
countermeasure are in fault?

## Sophos, the security vendor

[Sophos](http://www.sophos.com/en-us/) is a security vendor making products about end point protection, data and malware protection, compliance and last week it was [compromised too](http://www.sophos.com/en-us/lp/partnerportal).

From the the Sophos break-in report:

> Two unauthorized programs were found on the server, and our preliminary
> investigations indicate that these were designed to allow unauthorized remote
> access to information.

What we can't understand from the official statement is if the vulnerability
was in the web application managing the partner's portal or in some daemons
allowing attackers to breach the system.

However it was possible by the attackers to upload and run binaries. So there
are no antivirus/antimalware protection on the server or it was a 0 day malware
binary not detected by Sophos. Since the vendor makes such products, it would
be great having more information on what it was uploaded on the server. 

The result however is that disclosed information are critical:

> Data included in the server's databases include: Partners' names and business
> addresses, email addresses, contact details, and hashed passwords.

## Some conclusion

Breaches occur. There is no silver bullet to avoid attacker to breach you
system and you must no thinking _"it will never happen to me"_.

Also if you don't manage credit card numbers or other sensitive data that can
be sold, attackers can be interested in owning your systems to use them as
attacker launch platform.

You must start thinking _"I must pay attention to avoid that this can happen to
me"_ if you want to play nice in the IT place. This is about being professional
and protect your users' data.

You must check your system for security issues with both vulnerability
assessment than web application penetration test, before system hit the Net.
This is to make sure that at least all the patches are applied and the web
application is strong and good written.

You must segregate your database layer in a private network different from the
DMZ where the webapp lays and you must protect database access using firewalls
and access lists.

Finally, if in your organization a security breach occurs it will be great
disclose all the information about who accessed which data and how.

You must tell your users the security checks you made just to prove that
attackers were highly skilled and the breach was inevitable and you must tell
the mitigation roadmap and which are the countermeasure acts you want to put in
place to avoid the same thing will happen again.
