---
layout: post
title: "When you realize you're doing threat modeling"
date: 2012-09-18 08:41
comments: true
published: true
featured: false
categories: builders wapt appsec threat-model threat questionnaire
hn: 
rd: 
thumb: t-model.jpg
---
Yesterday I was in a meeting for an appsec activity about a legacy PHP web
application. In front of my a couple of experienced developers with an in-deep
knowledge of their code and their architecture (and sometimes this is the good
news of the day).

In a friendly way we started enumerating possible web application access
scenario, through VPN or via servers in segregated LANs.

We started a great threat modeling jam session.

<!-- more -->

When starting an application security activity over a web application you can
for sure start by trying to break the code or the system it was running it.

If you want to act like a pro you may want to start engaging developers and
architects asking them to tell you about their code, how users they will
interact, which protocols will they use.

The key concept here is the you must enumerate all the possible entry points in
your web application in order to evaluate where a possible attacker can coming
from. This is true also for Internet exposed web applications, in this case
other than the web you must enumerate all possible service doors an attacker
can use to break your system.

After that you must ensure that communication protocols are consistent and
robust. You must ask confirmation about cryptography to be used for logon pages
and you must ask details about who enrolled the digital certificate, if it is
self signed, signed by an internal CA for intranet apps or if it is enrolled by
a trusted certification authority.

**Hint:** you may want to use
[ciphersurfer](https://github.com/thesp0nge/ciphersurfer) to check your website
for SSL robustness. A score of B is the low level bound for a production
server.

It's really important that if your web application exposes some APIs you
enumerate all the services, with the parameters and their value and the HTTP
verb you must use to invoke them. The idea is that with this information you
can start testing for unexpected conditions making sure the APIs is robust and
consistent and that it gracefully manage errors or out of bounds parameters.

{% blockquote %}
You must enumerate all the possible entry points in your web application in
order to evaluate where a possible attacker can coming from.
{% endblockquote %}

In yesterday's scenario it was clear after the informal meetup that some of the
tests I had in mind were completely pointless and instead I discovered new
possible attack scenarios I won't be aware of without this session.

Of course good threat modeling works if the developers and the architectures in
front of view are skilled enough to help you in looking at the whole picture
and sometimes this is not true.

For a more structured and formal Threat modeling session you may want to look
at the [OpenSAMM project](http://www.opensamm.org/).
