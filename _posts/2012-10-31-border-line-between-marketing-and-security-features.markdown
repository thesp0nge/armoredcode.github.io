---
layout: post
title: "Border line between marketing and security features"
date: 2012-11-05 09:00
comments: true
published: true
featured: false
categories: breakers cross-site-scriting owasp owasp-top-10 xss ie xss-filter marketing security-feature attack-filtering h4f casper webrick ruby 
thumb: borderline.jpg
level:
hn: http://news.ycombinator.com/item?id=4742655
rd: http://redd.it/12nq1m
---

Make a web application penetration test is becoming tricky due modern browsers
native anti-xss filtering facilities _(they only work for reflected cross site
scripting)_.

Firefox and Google Chrome leave attack pattern in the resulting HTML code but
they don't render it. 

Microsoft Internet Explorer uses its internal [XSS Filter](http://blogs.msdn.com/b/ie/archive/2008/07/02/ie8-security-part-iv-the-xss-filter.aspx)
library.

This one can disabled server side by the web application.

<!-- more -->

To be honest, Microsoft is one of the IT companies that provides a lot of
documentation despite their anti-opensource crousades in the past.
In this [msdn blog post](http://blogs.msdn.com/b/ieinternals/archive/2011/01/31/controlling-the-internet-explorer-xss-filter-with-the-x-xss-protection-http-header.aspx)
it is explained how IE will react when it detects a reflected cross site scripting.
A [test page](http://www.enhanceie.com/test/xss/default.asp) is provided for
people who wants to play with this feature.

Since this page is vulnerable you can either check how the browser you're using
right now it behaves when a XSS is exploited under your feet.

Post author, Eric Lawrence, says in his post:

{% blockquote %}
Pages that have been secured against XSS via server-side logic may opt-out of
this protection using a HTTP response header: X-XSS-Protection: 0
{% endblockquote %}

Wait. Is it possible to disable the filter server side just with an HTTP header in the response?
Are you kidding at me?

## Too much power without control

Using HTTP response to disable browser security feature is a non sense, for a number of reasons:

* what if the server-side logic introduced to mitigate reflected cross site
  scripting is weak and it doesn't block some attack payload?
* what if new attack payloads have been discovered during the ages?
* what if the server-side logic can be either disabled or it's not used in some
  circumstances?
* what if a rogue HTTP response header it has been injected by an attacker to
  disable your client security filter?

## Let's exploit it

Do you remember [casper](https://github.com/thesp0nge/casper), the tiny
transparent proxy I wrote some months ago?
I first wrote this tool to quick check asynchronous calls a friend web site
made to evaluate performances.

casper extends
[WEBrick::HTTPProxyServer](http://www.ruby-doc.org/stdlib-1.9.3/libdoc/webrick/rdoc/WEBrick/HTTPProxyServer.html)
class. Making a transparent proxy this way is [trivial programming excercise](http://armoredcode.com/blog/h4f-invisible-proxy-dot-dot-dot-casper-gem/).

It's fun, but not that much. Let's add to casper some offensive super power.

In a [separated branch](https://github.com/thesp0nge/casper/tree/add_xss_ie_injector) I
implemented an injection code to disable XSS Filter protection for Microsoft
Internet Explorer.

The real magic is in the *lib/casper/disable\_ie\_xss\_protection.rb* source
code. I opened the WEBrick::HTTPResponse class back again and I added a
disable\_ie\_xss\_protection method playing with HTTP response headers sent
back to the browser.

``` ruby lib/casper/disable_ie_xss_protection.rb
class WEBrick::HTTPResponse

  def disable_ie_xss_protection
    self["X-XSS-Protection"]= 0
  end
  
end
``` 

Seven lines of code counting two empty lines I added for readability purposes.

In the Casper::Proxy class, I tell WEBrick to honor also response handlers in
the initialize method:

``` ruby lib/casper.rb
def initialize(config={})
  @req_count = 0
  @hosts=[]
  @urls=[]
  @trace_domain = ""
  @trace_domain = config[:trace] if config[:trace] and ! config[:trace].empty?

  config[:Port] = 8080 if ! config[:Port]
  config[:AccessLog] = []
  config[:ProxyContentHandler] = Proc.new do |req, res| 
    res.disable_ie_xss_protection if config[:disable_ie_xss_protection]
    log_requests(req, res) 
  end

  super(config)
end
```

I added a command line flag for casper script in order the user to turn the xss filtering avoidance on:
``` ruby bin/casper

opts = GetoptLong.new(
  [ '--help', '-h', GetoptLong::NO_ARGUMENT ],
  [ '--version', '-v', GetoptLong::NO_ARGUMENT ],
  [ '--trace', '-T', GetoptLong::REQUIRED_ARGUMENT],
  [ '--port', '-p', GetoptLong::REQUIRED_ARGUMENT ],
  [ '--disable-ie-xss', '-x', GetoptLong::NO_ARGUMENT ]
)

...

opts.each do |opt, arg|
  case opt
  
  ...
  
  when '--disable_ie_xss_protection'
    options[:disable_ie_xss_protection] = true
  end
end

``` 

## Off by one

Bringing browser to the next level adding security features it is a great move
from vendors. Adding the capability to turn off these security feature with a
server side HTTP response it is a **poor** decision, and I suspect it's more
marketing driven than a real users request.

On the other side with a bunch of ruby lines of code I turned
[casper](https://github.com/thesp0nge/casper) to be an attacking tool and this
is the direction I'm moving so far.

Some more advanced injection patterns will be added in the future and I will
integrate [casper](https://github.com/thesp0nge/casper) with
[cross](https://github.com/thesp0nge/cross),
[ciphersurfer](https://github.com/thesp0nge/ciphersurfer),
[links](https://github.com/thesp0nge/links) and other gems.

This melting pot will be the dynamic
[sake](https://github.com/codesake/sake)testing engine and it will be used with
static analysis engine I will release just for ruby and javascript languages
right now to create an hibryd security scanner.

If you want to read more about Cross Site Scripting, I suggest you to go to
[this Owasp.org page about this kind of attack](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS\))

Enjoy it!

_Borderline image credits to [Lynne Hand](http://www.flickr.com/photos/your_teacher/2595182363/)_  
