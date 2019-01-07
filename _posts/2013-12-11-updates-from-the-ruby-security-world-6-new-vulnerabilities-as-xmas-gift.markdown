---
layout: post
title: "Updates from the Ruby security world: 6 new vulnerabilities as X'mas gift"
date: 2013-12-11 08:01
comments: true
published: true
featured: false
tags: appsec ruby rails ruby-on-rails security xss dos denial-of-service sqli sql-injection builders codesake codesake-dawn dawnscanner cve-2013-4491 cve-2013-4492 cve-2013-6415 cve-2013-6414 cve-2013-6416 cve-2013-6417
thumb: light-rails.png
level:
hn: 
rd: 
---

Wow, last week it was very busy in the [ruby security annonuncement discussion
group](https://groups.google.com/forum/#!forum/ruby-security-ann). A bunch of
six new vulnerabilities were announced and, most of them, are cross site
scripting issues. This is bad for a problem floating around those places since
more than a decade.

<!-- more -->

## XSS on Simple Form

[Rafael Mendonça França](http://twitter.com/rafaelfranca) announced that
[Simple Form gem](http://rubygems.org/gems/simple_form) introduces a cross site scripting vulnerability in its
\1.1.1 version and beyond. Fixed releases are 3.0.1 and 2.1.1. 

The vulnerability appears _"when Simple Form creates a label, hint or error
message it marks the text as being HTML safe, even though it may contain HTML
tags. In applications where the text of these helpers can be provided by the
users, malicious values can be provided and Simple Form will mark it as safe."_

Rafael suggested also a quick-and-dirty workaround in case you can upgrade
Simple Form in your project. Escape input coming from the users:

``` ruby
  f.input :name, label: html_escape(params[:label])
```

You can read the original post
[here](https://groups.google.com/forum/#!topic/ruby-security-ann/flHbLMb07tE)

It seems there's no CVE identifier associated to this issue, I will track on
[codesake-dawn scanner](http://rubygems.org/gems/codesake-dawn) as
**Simple Form XSS - 20131129**.

## CVE-2013-4491 Reflected XSS Vulnerability in Ruby on Rails (via the i18n ruby gem that takes CVE-2013-4492)

Tenderlove announced on December 3rd a reflected cross site scripting
vulnerability affecting Rails but with a root cause in the i18n
internationalization gem. From the announcement: _"When the i18n gem is unable
to provide a translation for a given string, it creates a fallback HTML string.
Under certain common configurations this string can contain user input which
would allow an attacker to execute a reflective XSS attack."_.

This is huge and it affects of course all MVC ruby frameworks using that gem. I
guess that rails deserves a standalone CVE since the dependency tree can't be
changed in some circumstances.

You can read the original post [here](https://groups.google.com/forum/#!topic/ruby-security-ann/pLrh6DUw998)

## CVE-2013-6415 - XSS Vulnerability in number\_to\_currency

Same day, same forun, same kind of issue. A cross site scripting to the
number\_to\_currency helper. One of its parameter is not correctly escaped so
application which pass user controlled data as the unit parameter are
vulnerable to an XSS attack.

There is of course a workaround, using explicit escaping on the :unit
parameter:

``` ruby
<%= number_to_currency(1.02, unit: h(params[:currency])) %>
```
You can read the original post [here](https://groups.google.com/forum/#!topic/ruby-security-ann/9WiRn2nhfq0)

## CVE-2013-6414 - Denial of Service Vulnerability in Action View

There is a denial of service vulnerability in the header handling component of
Action View. Strings sent in specially crafted headers will be cached
indefinitely.  This can cause the cache to grow infinitely, which will
eventually consume all memory on the target machine, causing a denial of
service. 

As you can read on the original
[post](https://groups.google.com/forum/#!topic/ruby-security-ann/A-ebV4WxzKg),
there is a monkey patch you can apply if you can't upgrade rails.

``` ruby
ActiveSupport.on_load(:action_view) do
  ActionView::LookupContext::DetailsKey.class_eval do
    class << self
      alias :old_get :get

      def get(details)
        if details[:formats]
          details = details.dup
          syms    = Set.new Mime::SET.symbols
          details[:formats] = details[:formats].select { |v|
            syms.include? v
          }
        end
        old_get details
      end
    end
  end
end
```

You can read the original post [here](https://groups.google.com/forum/#!topic/ruby-security-ann/A-ebV4WxzKg)

## CVE-2013-6416 - XSS Vulnerability in simple\_format helper

Ruby on Rails simple\_format helper that converts user supplied text into html
text which is intended to be safe for display. A change  made to the
implementation of this helper means that any user provided HTML attributes will
not be escaped correctly.  As a result of this error, applications which pass
user-controlled data to be included as html attributes will be vulnerable to an
XSS attack.

A workaround is in place, but for the nature of the helper itself I personally
suggest to upgrade rails framework. It affects only 4.0.x framework version.

``` ruby
simple_format(some_text, class: h(params[:class]))
``` 

You can read the original post [here](https://groups.google.com/forum/#!topic/ruby-security-ann/A-ebV4WxzKg)

## CVE-2013-6417 - Incomplete fix to CVE-2013-0155 (Unsafe Query Generation Risk)

In January there was a big issue affecting Rails and SQL injection. For sure
[github.com](https://www.github.com) remind the [massive assign
vulnerability](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2013-0155)
that leads to sql injection allowing people to change data in backend
databases. 

The usage of third party libraries can lead the vulnerability remediation to be
circumvented.

You can read the original post [here](https://groups.google.com/forum/#!topic/ruby-security-ann/niK4drpSHT4)

## Off by one

As you may see, since Rails is gaining popularity vulnerabilities came in.
After all there is no 100% secure software here on Earth.

I'm working today and tomorrow to include those 6 security checks in
[codesake-dawn](https://github.com/codesake/codesake-dawn) and release a
version 0.80.

For doing this I had to postpone some custom XSS check against specific padrino
and sinatra application views, givin priority to the work done including CVE
framework related vulnerabilities.

Any question about the scanner you can make a tweet using #dawnscanner hashtag.
