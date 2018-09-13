---
layout: post
title: "LeakedIN and the salt and pepper sauce"
date: 2012-06-07 17:17
comments: true
published: true
featured: true
categories: breakers linkedin password information-leakage salt sha1 owasp 
hn: http://news.ycombinator.com/item?id=4083141
rd: http://redd.it/ur9ag
---

Two days ago, the Internet was squashed by a very large sensitive data breach.
More than 6.4M of password hashes coming from
[LinkedIN](http://www.linkedin.com) were published by an unknown attacker crew
exposing a large number of users to a credentials disclosure.
 
<!-- more -->

## It's the same old story

The funny side of all of this story is that it happened to a large website.
When such a breakins happened to small startups people think the root cause is
the cheap architecture underlying the web service.

Few money to spend means few investment in security assessments and code
reviews since all of money must be spent for business development.

But this is not an ideal world and a _big guy_ has been caught.
And he reacted as all _big guys_ telling something like:

{% blockquote %}
Yes we have a problem but hey... we sent an email to all of affected users to
change their password. Why happened? How the attacker break the site and stole
the information?

**plonk**
{% endblockquote %} 

No public statement about the incident and bear in mind that LinkedIN offers also a paid subscription service.
Is it fair not to publicly explain what's wrong happened?

After all, there is no code 100% secure.

## Salt, pepper and hashes

The archive contains SHA1 hashes and, as confirmed by LinkedIN, no salt was
used to perturb the clear-text obfuscation.

I see a lot of comments in Italian forums and on some international mailing
lists and more than 90% of people saying something was complaining about the
missin salt.

Is really this the bad programming practice to deprecate? 

Short answer: having the salt would help but some other bad things happened
before.

## The first line of defence

You've got a server (more likely a cluster of servers) published on the
Internet. You're a founded company serving tons of users and requests per day. 

1. *Segregation*. Your tech team is great and **obviously** it used a 3 layers
   architecture with a frontend layer exposed on a DMZ protected eventually by
   a web application firewall mitigating script kiddies attempt and by a perimeter
   firewall allowing inbound traffic only on the TCP port 80 and 443. Then you
   have a backend layer in the internal network connected with the frontend by a
   firewall and then the database layer connected with the backend by a third
   firewalling layer allowing only SQL traffic coming from backend and some SSH
   from a service VLAN.
2. *Patch management*. All involved servers have latest patches applied and
   they are constantly monitored for suspected behaviour or for hardware
   issues.
3. *Security scan*. Your security team has been periodically engaged to make
   vulnerability assessment, penetration tests, web application penetration
   tests and code review over the servers and the source code you published on the
   Internet and that is, let me to remember, your core business.

   You do offer a paid service, don't you?
4. *Security management*. Scan results brings you to have a risk level bounded
   to your architecture and you do have logs from appliances protecting your
   architectural perimeter and you have skilled people looking at this logs
   looking for anomalies or suspected attacks.
5. The source code. You allowed people to use "fuffy123" or "god" or "passedin"
   as password? Well the first step is to introduce some password complexity
   rules and force people to follow them.

It's clear that a break-in occured and an attackers reached the database and
stole information.

Do you agree with me that having the salt is one of the last problem?
It's clear that something is missing or it didn't work from a security point of view?

Was it a WAF failure? Was linkedin web application vulnerable to a SQL
injection? Was the web sites exposing a backup file cointaining the hashes?

I don't know but saying "hey linkedin, you must use SALT" is somewhat too
limiting if we want to suggest a way to expose a service with security in mind.

## The last line of defence: protect your data

However, all the stuff I said before have a cost. Buying appliances hardware,
software licences or have a skilled technical people do have a cost. 
You have to make sure to arrange some budget and implement _something_ to
protect the architectural perimeter.

Let's say however that a skilled attacking team decided you're an appealing
target, and linkedin site do is since it have personal accounts about IT and
non professionals and _all of us know_ that most of people do use the same
password everywhere.

Storing passwords in plain text is not an opinion I want to even consider but,
it's said to say, a lot of web sites I saw in these years store password
without encryption just relying on HTTPS as only security feature.

Using poor hashing algorithm like MD5 or SHA1 can be a solution for the
homemade blog engine serving a website talking about kittens or cooking or
guitar pedalboards. 

The starting up solution is to use SHA256 or even better SHA512 with a random salt of course.

Consider the following code working with ruby 1.9.2 or later (I don't check if it was present in earlier versions):

``` ruby a code snippet making a SHA256 of a password with a random salt
require 'digest/sha2'

File.read('/dev/urandom', 'r') do |f|
  @salt=f.read(20)
end

# Intermediate hashing the salt to have it in a some canonical way 
# It's not an obligatory step...

salt_digest = Digest::SHA256(@salt)

pass_digest = Digest::SHA256(salt_digest+pass_digest)
``` 

The important part of all of this is that you **must save** your salt somewhere
in order to check a cleartext password against the stored digest.

Do you like it?

Well, true to be told the solution I like most is to use bcrypt to make a password digest.

``` ruby a code snipped with bcrypt
require 'bcrypt'

class User < ActiveRecord::Base
  # users.password_hash in the database is a :string
  include BCrypt

  def password
    @password ||= Password.new(password_hash)
  end

  def password=(new_password)
    @password = Password.create(new_password)
    self.password_hash = @password
  end
end
``` 

Applying this to other ORMs is a trivial programming exercise.

### Update

A reader who wants to stay in the shadows reported a broken link and the need
to have a bcrypt driven example like the one using SHA256.

Here it is:

``` ruby creting a digest with bcrypt
require 'bcrypt'

my_password = BCrypt::Password.create("my password") #  => "$2a$10$uwmjwR5B6JpzQ9luq7fAt.U/xpDl9c9EU/EQS8hIJQqEzEivOq3S." 
``` 

## End of file

Applying a SALT to a password before hashing the value is a good practice of
course. SALT **must** be chosen in a safe way, "1234salt" is not a good one
"e0571fd31a7aa9dce757ecd33868ba04" can be ok for must of cases,
"636c895647b184e4837d0d9b72b8336aa6e636c7e42d5fc3b8ead15d8464ccf0" is a killing
choice for _I want to forget myself about people using rainbow table to guess
the salt_.

Moreover if you read the SALT from a truly random source like /dev/urandom in
Unix systems.

Using bcrypt makes you safe even from attackers using rainbow table and
bruteforce attacks. Remember that you must think yourself in the worst
situation when you have to make a security architectural choice.
The worst situation is _the attacker has both digest and salt, how can I
prevent him to reverse and break the account?_

{% blockquote %}
You must think yourself in the worst situation when you have to make a security
architectural choice.
{% endblockquote %}

Choosing bcrypt solves the question since it's resilient to bruteforce attacks and it's almost safe from reversing.
Even devise [switched to bcrypt](http://groups.google.com/group/plataformatec-devise/browse_thread/thread/60b69148899ee94a?pli=1) 
almost two years ago or something.
