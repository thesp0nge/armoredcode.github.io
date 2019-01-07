---
layout: post
title: "Rails patching Wednesday"
date: 2014-02-19 07:49
comments: true
published: true
featured: false
tags: rails ruby-on-rails xss sql-injection cve-2014-0080 cve-2014-0081 cve-2014-0082 security builders
image: 
level:
hn: 
rd: 
---

Past weeks were busy for Ruby on Rails core team and appsec people looking at
the framework's security. Yesterday, core rails member [Aaron
Patterson](https://twitter.com/tenderlove) announced three Ruby on Rails
security issues affecting latest versions and obviously all the web
applications out there built on affected issues.

<!-- more -->

## CVE-2014-0080: Data Injection Vulnerability in Active Record

Looking at the vulnerability announcement, it seems a SQL Injection is possible
when RoR relies on PostgreSQL as backend database. Specially crafted strings
can be used to save data in PostgreSQL array columns that may not be intended.
Exploiting this vulnerability doesn't allow the attacker to delete data or
execute arbitrary SQL statements, however due to its nature, it allows the
attacker to add arbitrary data. This can have impact on the application
behaviour.

Affected release are 4.0.x and 4.1.0.beta1. It seems no previous version is
affected by this one.

If you can't upgrade, you can use the monkey patch provided by [@tenderlove](https://twitter.com/tenderlove)

```ruby
module ActiveRecord
  module ConnectionAdapters
    class PostgreSQLColumn
      module Cast
        alias :old_quote_and_escape :quote_and_escape

        ARRAY_ESCAPE = "\\" * 2 * 2 # escape the backslash twice for PG arrays
        def quote_and_escape(value)
          case value
          when "NULL", Numeric
            value
          else
            value = value.gsub(/\\/, ARRAY_ESCAPE)
            value.gsub!(/"/,"\\\"")
            "\"#{value}\""
          end
        end
      end
    end
  end
end
```

## CVE-2014-0081: XSS Vulnerability in number\_to\_currency, number\_to\_percentage and number\_to\_human

This is the second time a Cross Site Scripting vulnerability is spotted in
formatting helpers for numbers. Some of the parameters to the helper (format,
negative\_format and units) are not escaped correctly. Application which pass
user controlled data as one of these parameters are vulnerable to an XSS
attack.

As a workaround, but please note that upgrading rails gem is strongly
recommended, you can use the ```h``` sanitizing helper to filter it out user
controlled strings.

```
<%= number_to_currency(1.02, format: h(params[:format])) %>
```

For my personal taste, making defensive programming and filtering user
controlled inputs is **always** the way to do.

## CVE-2014-0082: Denial of Service Vulnerability in Action View when using render :text

There is a denial of service vulnerability in the text rendering component of
Action View. Strings sent in specially crafted headers will be converted to
symbols. This can cause a denial of service since symbols are not removed by
the garbage collector.

There is no workaround here, upgrade it

## Upgrade upgrade upgrade

In order for you to fix all the aforementioned vulnerabilities you have to
upgrade rails gem to versions 4.1.0.beta2, 4.0.3 and 3.2.17. For people stuck
at version 3.1.x and 3.0.x there is no solution to CVE-2014-0081 and
CVE-2014-0082. Also if CVE-2014-0080 doesn't affect rails version previous than
4.0.0 it is not enough for a programmer to say "I'll stay on my 3.1.2 because
it's comfortable". Please go ahead with a migration plan.

## Off by one

[Brakeman](http://brakemanscanner.org) already have check for CVE-2014-0080 in
its github repository and [Codesake::Dawn](http://dawn.codesake.com) will
include all those three security checks in upcoming version 1.1.0.

Please don't stuck on older framework versions just for sake of being stable.
This is a nonsense (like of course the opposite behaviour). Stay upgraded and
when your framework version tree is marked as deprecated, please move on or
you're exposing your customers (and your business too) to security risks.

Enjoy it!
