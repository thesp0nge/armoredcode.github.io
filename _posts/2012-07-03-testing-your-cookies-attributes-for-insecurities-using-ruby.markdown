---
layout: post
title: "Testing your cookie's attributes for insecurities using ruby"
date: 2012-07-05 07:42
comments: true
published: true
featured: true
categories: builders breakers ruby webapp gems pentest speech rubyday owasp owasp-testing-guide pentest-with-ruby
hn: http://news.ycombinator.com/item?id=4202206
rd: http://redd.it/w2i0s
---

Session cookies are a swiss army knife for every developer to maintain user
session requests tracking. They need however to be designed with security in
mind since they can be used to claim an authenticated session by an attacker.

Let's see how to use [ruby](http://ruby-lang.org/en) to automate a test for
your cookie security configuration.

<!-- more -->

## A good testing framework underneath

As we see in other [pentesting with ruby](http://armoredcode.com/blog/categories/pentest-with-ruby/) posts serie,
the [Owasp Testing Guide](https://www.owasp.org/index.php/OWASP_Testing_Guide_v3_Table_of_Contents) is an
outstanding place to start to find inspiration for the security tests to
implement.

The paragraph 4.5 of the guide describes what to test for [session management](https://www.owasp.org/index.php/Testing_for_Session_Management).

In this post we want to focus on the
[OWASP-SM-002](https://www.owasp.org/index.php/Testing_for_cookies_attributes_(OWASP-SM-002\))
check, testing for cookies attributes.

## A recall on cookies

You deployed a great web application with a custom authentication backend.

Your users input their credentials in a HTTPS session, you validate the data
they sent and you give back a session identifier cookie to be used in
subsequent requests.

A cookie is a variable your browser is instructed to save somewhere in memory
(or in the filesystem for **persistent cookies**) and that it must give back to
the server everytime you make a request upon logout.

Web application make a server side validation for the cookie your browser is
submitting. If it's a valid cookie, you're in otherwise you'll be redirected to
the logout page and your session will be invalidated (it should hopefully work
this way for every web application on the planet, but sometimes it doesn't work
this way).

Logout procedure is not something _magical_. User session is either invalidated
both server side by relasing all the resources your web application previously
allocated and client side by telling the browser to delete the cookie you
assigned.

## The cookie attributes

<table class="table-striped table-bordered table">
  <thead>
    <tr>
      <th>Attribute</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>secure</td>
      <td>
        if set, the browser is forced to transmit this cookie **only** over a
        secure protocol like HTTPS. This is the first line of defence to avoid cookies
        evesdrop.
      </td>
    </tr>
    <tr>
      <td>HttpOnly</td>
      <td>
        to avoid cookie to be steal using javascript (e.g. in a cross site
        scripting attack vector), you can set this attribute forcing the browser to
        transmit the cookie <strong>only</strong> using HTTP protocol family.
      </td>
    </tr>
    <tr>
      <td>domain</td>
      <td>
        makes the cookie to be valid server side only for that domain. Please
        note that only hosts from a particular domain can create cookies for their own
        domain.
        
        There are some restrictions for applications served at different sub
        domains but we won't go deeper for this attribute at this stage.
      </td>
    </tr>
    <tr>
      <td>path</td>
      <td>
        if set, this attribute can be used with the <em>domain</em> attribute to make
        sure a cookie is valid for a particular path and it won't be valid for other
        paths in the same web applications. Can be useful to restrict an online shop
        cookie to be bound on the checkout url.
      </td>
    </tr>
    <tr>
      <td>expires</td>
      <td>
        if set, this cookie is called <strong>persistent</strong> and it will be saved to
        filesystem and reused by the browser until the expire date will be over. Please
        note that there is a <em>UE</em> regulamentation that says a website is supposed to
        declare the users if the cookies used are persistent or volatile, mainly it is
        done for privacy reason to make aware people that they must delete cookies from
        a shared computer if they want to protect their privacy.

        As a lot of laws about the Internet, this one it's not that clear and at least
        here in Italy can be easily misunderstood. 

        The important bit is that storing a sensitive cookie on filesystem <strong>is</strong> a
        security issue. When in doubt, don't use persistent cookie.
      </td>
    </tr>
  </tbody>
</table>
  
## Implement OWASP-SM-002 with ruby

### Obtain a cookie within a session

We will use [mechanize](http://mechanize.rubyforge.org/) gem to automate
interactions with our target web application, as we made in
[cross](https://github.com/thesp0nge/cross).

Tests we will make here are not invasive so they can easily replicated to every
web application we use everyday. We just automate logon using our credentials
and evaluate the cookie the website it will set.

Please note that trying to guess other users account can be interpreted as an
offensive attempt.

``` ruby an simple basic authenticatiokn login using mechanize
# as seen here with some corrections: http://stackoverflow.com/questions/5291576/basic-and-form-authentication-with-mechanize-ruby
require 'mechanize'
require 'logger'

agent = Mechanize.new { |a| a.log = Logger.new("mech.log") }
agent.user_agent_alias = 'Windows Mozilla'
agent.auth('username', 'password')
agent.get('http://example.com')
cookies = agent.cookies
```

### Testing for the secure flag

``` ruby testing for cookie's secure attribute
cookies.each do |c|
  puts "Cookie #{c.name} is not declared as secure" if ! c.secure?
end
``` 

### Testing for the HttpOnly

It seems that [original mechanize](https://github.com/tenderlove/mechanize)
form Tenderlove doesn't support HttpOnly, that's why I forked the repo and I'll
add asap to add an httponly attribute for cookie.

For such a reason I forked original repository creating [a branch](https://github.com/thesp0nge/mechanize/tree/add_httponly_cookie_value)
where I added an httponly accessor attribute.

Testing for HttpOnly declared cookies is so very simple. 
Please note that the test bed I added to mechanize fails and I don't know why.
Debugging a little further it seems that httponly test is called twice, the
first time succeeded and the second one it fails. 

I will investigate further on this but in meantime you can dig it and help me
finding the caveat.

``` ruby testing for cookie's httponly attribute, you must use my fork of mechanize 
cookies.each do |c|
  puts "Cookie #{c.name} is not declared as HttpOnly" if ! c.httponly?
end
``` 

### Combining the two aforementioned tests

The two tests we just saw aren't so meaningful if taken divided. Let's combine
them in a single loop telling that the declared cookie can't be hijacked due to
HttpOnly flag and it can't be tampered by an in the middle attacker due to SSL
protection.

``` ruby testing for cookie's resilience to hijack and tamper. Please note you must use my fork of mechanize 
cookies.each do |c|
  puts "Cookie #{c.name} is not secured" if ! c.httponly? and ! c.secure?
end
``` 

### Testing for the domain

For a security perspective, the domain value must not be choosen loosely in a
way a cookie can be either good for other servers in the domain.

As example we can consider an application served over
http://shop.armoredcode.com (it doesn't exist, don't check! =)). If we set a
cookie valid for the domain armoredcode.com containing some secrets, every host
in the domain can use this cookie.

This means that if http://bugged.armoredcode.com is underattack, cookies coming
from my shop site can be steal by an attacker using another web application of
my own domain.

Pretty scary, isn't it?!? Well, fixing this issue it's easy, cookie domain must
be set for the host the cookie it's intended to be.

Checking it's also straightforward easy as well, we can use the [API Mechanize](http://mechanize.rubyforge.org/Mechanize/Cookie.html)
(the original repo) give use.

``` ruby testing for cookie domain not being too loose
cookies.each do |c|
  puts "Cookie #{c.name} has an insecure domain configuration" if c.for_domain?
end
``` 

### Testing for the path

Path attribute must not loosely configured to match against '/' pretending that
all pages and all even other applications on the same server can access this
cookie. Cookie path must match the path of the web application we asked as
target.

``` ruby testing for the path attribute 
require 'uri'

target="http://www.mygreatapp.com/shop"
uri = URI.parse(target)

agent = Mechanize.new { |a| a.log = Logger.new("mech.log") }
agent.user_agent_alias = 'Windows Mozilla'
agent.auth('username', 'password')
agent.get(target)
cookies = agent.cookies

cookies.each do |c|
  # we must add a trailing / to the path since the cookie path attribute must be set with it by the server
  puts "Cookie #{c.name} has an insecure path value" if uri.path+'/' != c.path
end
``` 

### Testing for the cookie being expired

A cookie must be valid in time and we cna use [Mechanize API](http://mechanize.rubyforge.org/Mechanize/Cookie.html) to check for the cookie being valid.
However I w like to issue a warning if the expire value has been set in order
to prompt that the cookie it's intended to be persistent.

``` ruby testing for the cookie expire value
cookies.each do |c|
  puts "Cookie #{c.name} is expired" if c.expired?
  puts "Cookie #{c.name} is persistent" if ! c.expired.nil?
end
``` 

## Merging all together

Running those tests we can evaluate how much each cookie we declared fulfill
[OWASP-SM-002](https://www.owasp.org/index.php/Testing_for_cookies_attributes_(OWASP-SM-002\))
and then it can be considered secure enough to store our customer secrets like
session id or shop basket purchases. 

However it would be nice telling how much good is the cookie assigning a score
to each test giving an overall value at the end.

We can say, that the cookie it must be secure and httponly is the wanted
baseline. So we can make a weighted average giving 2 points each other and just
a 1 point for all the other three tests.
