---
layout: post
title: "Happy birthday armoredcode and 4 rails advisories"
date: 2013-03-18 18:45
comments: true
published: true
featured: false
tags: birthday rails ruby-on-rails tenderlove builders breakers xss cve-2013-1855 cve-2013-1857
thumb: cake.png
level:
hn:
rd:
---

It was [a year ago](http://armoredcode.com/blog/hello-world/) when I started
the [armoredcode.com](http://armoredcode.com) project.

The goal, it's useful to recall it, is to talk to developers about application
security. And this evening there are three new security advisories for
[the Ruby on Rails MVC framework](http://rubyonrails.org).

<!-- more -->

## Is rails under attack?

Yesterday, [@tenderlove](http://tenderlovemaking.com/) reported 4 new security
advisories for [Ruby on Rails](http://rubyonrails.org). Two of them are
advisories about Cross site scripting vulnerabilities affecting sanitize
helpers, therefore it's critical to upgrade to the latest rails version.


* [CVE-2013-1855](https://groups.google.com/d/msg/rubyonrails-security/4_QHo4BqnN8/_RrdfKk12I4J)
  XSS vulnerability in sanitize_css in Action Pack
* [CVE-2013-1857](https://groups.google.com/d/msg/rubyonrails-security/zAAU7vGTPvI/1vZDWXqBuXgJ)
  XSS Vulnerability in the `sanitize` helper of Ruby on Rails

Tenderlove also gave some monkey patch to apply in case you can't patch your
rails installation.

``` ruby @tenderlove monkey patch for CVE-2013-1855
module HTML
  class WhiteListSanitizer
      # Sanitizes a block of css code. Used by #sanitize when it comes across a style attribute
    def sanitize_css(style)
      # disallow urls
      style = style.to_s.gsub(/url\s*\(\s*[^\s)]+?\s*\)\s*/, ' ')

      # gauntlet
      if style !~ /\A([:,;#%.\sa-zA-Z0-9!]|\w-\w|\'[\s\w]+\'|\"[\s\w]+\"|\([\d,\s]+\))*\z/ ||
          style !~ /\A(\s*[-\w]+\s*:\s*[^:;]*(;|$)\s*)*\z/
        return ''
      end

      clean = []
      style.scan(/([-\w]+)\s*:\s*([^:;]*)/) do |prop,val|
        if allowed_css_properties.include?(prop.downcase)
          clean <<  prop + ': ' + val + ';'
        elsif shorthand_css_properties.include?(prop.split('-')[0].downcase)
          unless val.split().any? do |keyword|
            !allowed_css_keywords.include?(keyword) &&
              keyword !~ /\A(#[0-9a-f]+|rgb\(\d+%?,\d*%?,?\d*%?\)?|\d{0,2}\.?\d{0,2}(cm|em|ex|in|mm|pc|pt|px|%|,|\))?)\z/
          end
            clean << prop + ': ' + val + ';'
          end
        end
      end
      clean.join(' ')
    end
  end
end
```

```ruby @tenderlove code to place into a file in config/initialized to fix CVE-2013-1857
module HTML
    class WhiteListSanitizer
      self.protocol_separator = /:|(&#0*58)|(&#x70)|(&#x0*3a)|(%|&#37;)3A/i

      def contains_bad_protocols?(attr_name, value)
        uri_attributes.include?(attr_name) &&
        (value =~ /(^[^\/:]*):|(&#0*58)|(&#x70)|(&#x0*3a)|(%|&#37;)3A/i && !allowed_protocols.include?(value.split(protocol_separator).first.downcase.strip))
      end
    end
  end
```

True to be told, I see the high number of security issues for Rails as a
symptom that the framework is gaining in popularity.

## The cross site scripting menace

It's one of the widespread security vulnerabilities affecting web applications
nowadays. For the [Owasp project](http://www.owasp.org) is one of the most
prevalent security risks for enterprises, in the 2010 it was the second item in
their Top 10 and in the 2013 it's going to be third in this security risks
standing.

It looses a place but it's far from being mitigated.

> The idea behind cross site scripting it's easy. A web application is vulnerable
> if it takes an input (either from a web form or from HTTP request) and it uses
> it without proper validation or escape.

There are three different type of cross site scripting (aka XSS) attacks:

* reflected
* stored
* DOM based

### Reflected cross site scripting

A reflected cross site scripting occurs when a web application consumes a user
input using it in one of its pages. A common scenario is a search result page
when the search key is echoed to recall the user about his choice.

Following there is a [Sinatra](http://sinatrarb.org) web application that takes
a parameter from the URL and it uses in the view to say hello to the user.
Of course this application is far from being something to be used in production
but it can be used to exploit a reflected XSS.

``` ruby a vulnerable Hello World Sinatra application
require 'sinatra'

get '/' do
  @name = params[:name]
  erb :index
end

__END__

@@index

<!DOCTYPE html>

<html>
  <head>
    <meta charset="UTF-8">
    <title>XSS test</title>
  </head>
  <body>
    <h1>Worked!</h1>
    <p>
      Hello <%= @name %>
    </p>
  </body>
</html>

```

Using with a regular string, this is the output we expect.
![]({{site.url}}/images/reflected_xss_notaint.png)

Since we **trust user inputs** (and this is a typical habit in software
development) if the name parameter is filled with this piece of js

  &lt;script&gt;alert('xss')&lt;/script&gt;

then the resulting HTML will be

``` html resulting body snippet
  <body>
    <h1>Worked!</h1>
    <p>
      Hello <script>alert('xss')</script>
    </p>
  </body>
```

Browser doesn't know it should receive a user name with an hello message. It
takes this piece of HTML and it renders it with the following result:

![]({{site.url}}/images/reflected_xss_taint.png)

In order to understand how dangerous it can be a reflected cross site
scripting, look at this scenario:

* a web site is vulnerable to reflected XSS
* an attacker designs a well crafted phishing email exploiting the XSS in a
  email link
* the attack pattern is a redirect to an attacker controlled website with a web
  page reading all cookies
* the user is redirected back to the vulnerable web site

``` javascript a cookie reading function
function get_cookies_array() {

    var cookies = { };

    if (document.cookie && document.cookie != '') {
        var split = document.cookie.split(';');
        for (var i = 0; i < split.length; i++) {
            var name_value = split[i].split("=");
            name_value[0] = name_value[0].replace(/^ /, '');
            cookies[decodeURIComponent(name_value[0])] = decodeURIComponent(name_value[1]);
        }
    }

    return cookies;

}
```

### Stored cross site scripting

Stored cross site scriptings occur when a vulnerable web application stores
user input in a database or a file for a further usage.

In a past web application penetration test, with a source code review, I found
this:

* a web application exposes a page for user feedback with a textarea html
  element in it
* web application saves users' feedback in the database without filtering or
  sanitize them
* an internal web application read that database for datamining activities

It was possible to submit, as fake feedback, a cross site scripting pattern
attack that is eventually stored in the database. When the second web
application (that is not Internet exposed) reads that database trying to build
a table with users' feedback, the pattern is used and the attack exploited.

That's a stored cross site scripting in brief.

### DOM Based cross site scripting

A DOM based xss attack occurs when the attack payload is executed by modifiying
the DOM document in the victim browser using an attack pattern executed client
side.

Richer user interfaces perform a lot of task client side, using parameters to
build forms or mask or populate values without passing such parameters to the
application server. This is done to save some traffic request and achieve
better performances.

If DOM is updated without filtering the values read by the user, then it's
possible to inject arbitrary javascript code that it will be executed client
side. Bear in mind that since no code is passed to the server, it's unlikely
that a web application firewall will save you from this kind of attack.

## Happy birthday armoredcode.com

In this first year of blogging:

* 24.701 people visited [this site](http://armoredcode.com) - unique visitors
* 74,74% visitors were English speaking people. Only 2.85% used their browser with Italian locale
* USA with the 26,37% of visits is the country that loves more armoredcode.com followed by Italy (10.04%).
* Very few of my visitors use Internet Explorer (4.11%) but we've a lot of Windows visitors out of there (44.93%

To all of you... **thank you very much**
