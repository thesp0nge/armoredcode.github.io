---
layout: post
title: "Bypassing HTTP Basic Authentication in PHP applications"
date: 2012-04-26 07:59
comments: true
published: true
featured: false
tags: breakers php basic-authentication design 
hn: http://news.ycombinator.com/item?id=3892897
rd: http://redd.it/st9nc
---

## Basic authentication doesn't work

Using HTTP basic authentication to protect backends or adminitrative panels is
a **bad** idea. Of course, setting up HTTP Basic auth for the web server you
live most is a trivial configuration exercise, however this approach bring
himself the following pitfalls:

1. HTTP Basic Authentication scheme doesn't offer a strong cryptographic system
   to protect your password. In fact the password is no more than encoded in
   Bas64 and reverting it is a trivial joke;
2. Most of times HBA (let's make the subject shorter since I will use it a
   lot), is served over plain HTTP session so your credentials will be
   transmitted in plain text over the wire;
3. HBA **can be bypassed** for applications written in PHP.

<!-- more -->

## It all started from a friend asking me...

Several months ago, a friend of mine came to me asking to make a _soft_
assessment over a web applications he was going to publish just for fun. 

> Please note that I was authorized by my friend making some crafted requests by
> the application owner. Any unauthorized attack is a criminal act that, at
> least here in Italy but almost all arount the world, can send you in real
> troubles so please, use the following techniques for testing and research
> purposes only.
> 
> While at work, use them over your customer websites after they give you a
> formal authorization and just on testing environment.
> 
> Application security is not about ruining other people work.

## Fingerprinting

Fingerprinting a web application and the backend technologies it uses is not
difficult. In example you can start from [netcraft](http://www.netcraft.com) to
ask information about the website even making a single HTTP request.

If you're not lucky enought with netcraft, you can make an HEAD request and look
for Server string in HTTP response

Let's use venerable netcat for this purpose:

``` 
echo -e "HEAD / HTTP/1.0\n\n" | nc yourtarget 80
``` 

Netcraft was enought to me. Target was a Ubuntu Linux distribution with Apache
and PHP. Using the apache version I was able to retrieve the Ubuntu version
shipped with the target. Just searching the deb packet each version shipped. A
2 yo cold... maybe even unpatched.

## A well designed MVC application failing miserably

Target web application was a shiny web 2.0 webapp full of javascript. Looking
at the JS, almost every link was crafted using REST and APIs were well designed
and consistent.

Apache was configured to route all 404 errors to main page with a 301 _Moved
permanently_ error code. This is the best choice in term of SEO to close your
web site in the links you are supposed to expose to the world.

At least, this is good if you don't forget to disallow access to a _asset_
directory.

I was aware that a "/asset" url was there since all js and css files but I
found the dir was even browsable exposing backend admin specific javascripts.
Looking at the source, the same url building policy was followed, so I
discovered where to request the website backend just because in javascripts
they gave me most of the backend urls.

Pointing the browser to "/backend", HBA popup appears. I
tried some common combination using also information about developers or web
agency I was able to gather from the Internet, but none of them worked.

I was close to giving up, however happy that I had something to report to my
friend. At the last chance I fired up the best hacking tool ever...
[google](http://google.com)

## A 2 years old problem

After googling a bit, I discovered 2 posts talking about HBA and PHP. The
[first post](http://cd34.com/blog/web-security/hackers-bypass-htaccess-security-by-using-gets-rather-than-get/)
saw it was hacked by an attacker that uses a non existent HTTP verb for
requests. It saw that Zend framework did honor the malformed verb, serving the
HBA protected PHP script exposing his webapp backedn.

In the comments I found another
[researcher](http://eguaj.tumblr.com/post/2361187940/re-hackers-bypass-htaccess-security-by-using-gets)
saying it was not a Zend specific vulnerability but a PHP 5.3 specific
behaviour. 

It seems that PHP to allow WebDav specific verbs, doesn't care about custom
verbs letting the script the choice about how to behave.

## A by design flaw

I won't say PHP has a security flaw since leaving the final decision up to the
script is reasonable. The real problem here is that authentication token is not
checked in the target script that it assumes that it's called by a legal user
that knows protected area password.

Owasp called this: Failure to restrict url access and it is the [8<sup>th</sup> voice](https://www.owasp.org/index.php/Top_10_2010-A8-Failure_to_Restrict_URL_Access) for the Owasp Top 10 - 2010.

## Let's exploit it

Crafting HTTP requests using a non existent verb can be boring using _echo_ and
_netcat_.
I want to automate the process allowing me to ask for specific backend sections
and routines, that's why I wrote a custom ruby script.

Net::HTTP ruby API is great. In order to create a custom non existent verb, all
you have to do is extending the Net::HTTPRequest and your client is ready to
fire up requests.

``` ruby extending the Net::HTTPRequest
require 'net/http'

class Dammi < Net::HTTPRequest
  METHOD="DAMMI"
  REQUEST_HAS_BODY = false
  RESPONSE_HAS_BODY = true
end
```

This way, we create a Dammi (_'dammi' is the Italian word that means 'give me'_) class
that introduces a DAMMI method that it doesn't have a body (so it will act like
regular GETs) but it will cause the server to have a body.

Next step is to open an HTTP client that we will use to fire our DAMMI requests.

``` ruby opening an http client to our target
http=Net::HTTP.new('www.mytarget.nonexistent', 80)
```

Now we are ready to go... we will simply make some DAMMI requests saving the
HTML response looking for XSS or SQL injection or other vulnerabilities.

In the real world attack I asked for the backend index.php page and I studied
the output HTML code to understand how the backend was made. Remember that it
was a blackbox testing and the backend was custom made so I would only assumed
that a index.php was in place.

``` ruby asking for index.php and putting response
r_a = http.request(Dammi.new("/backend/index.php"))
puts r_a.body
``` 

In the output HTML I saw that there were no other scripts but index.php that it
was called with different paramaters make some database updates over users
records.

I was very lucky since all the updates were driven by GETs with a special
parameter saying the backend it has to update some records.

If this was not the case, I had to write another custom HTTP verb with a body,
passing all the parameters in the body like a real POST.

That's all folks. My friend was not so happy since I gave him a lot of
remediation tasks that make his application to hit the web a month later than
the planned rollout date. I was happy I saved my friend works.

A matter of different points of view.

## Recap

I help my friend to save his work from attackers this way:

1. fingerprint the operating system, the web server and the programming
   language version using [netcraft](http://www.netcraft.com)
2. I discovered an "/backend" directory looking into a javascript file I found in
   a browsable "/static" directory.
3. I crafted custom HTTP requests in order to bypass HTTP Basic Authentication
   that it was in place to avoid curious people to look into the backend
4. I was able to make updates into the database... and I found also some cross
   site scripting vulnerabilities too.

## What we developers have to learn

1. Backends must be strongly protected by a login form with username and
   passwords
2. Passwords **must not** be saved in plaintext but encrypted using SHA-256 or
   SHA-512
3. Login forms must be served over an HTTPS connection
4. Your server side scripts must **always** check for authentication token in
   the HTTP session or wherever you choose to save it
5. Never trust users.

## Let's see it from the sysadmin point of view

** UPDATED, 27 April **

A lot of people are making comments about how to mitigate this vulnerability
seeing it as an Apache configuration flow.

This has a lot of sense. I must point out that no all sysadmins lockdown
.htaccess file in order to limit HTTP Verbs but the allowed ones.

For such a reason I would recall that you can use the [LimitExcept directive](http://httpd.apache.org/docs/2.2/mod/core.html#limitexcept) to lock
down the restricted area to all the tampered verbs but the ones you want to
allow access.

To mitigate the HTTP verb tampering attack from a sysadmin point of view, you
may want to allow only GET and POST verbs for the restricted area you want to
protect.
