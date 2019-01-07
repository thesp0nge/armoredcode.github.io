---
layout: post
title: "Driven by real world task: code reviewing JSP using regular expressions"
date: 2012-12-03 18:45
comments: true
published: true
featured: false
tags: builders jsp ruby sake code-review sast codesake application-security appsec
thumb: regex.png
level:
hn: http://news.ycombinator.com/item?id=4870052
rd: http://redd.it/1495ri
---

Nothing but solving a real world problem can help boosting a piece of software
to evolve.

In those days I'm engaged on a big Java written source code review. I submitted
the code onto the commercial tool we use to scan very wide codebases but, since
this tool output doesn't impress me much, I start searching the web for a Java
parser I can
integrate quickly in my [Owasp Orizon tool](http://armoredcode.com/blog/untold-owasp-orizon-is-died-and-im-sad-of-it/).

Unsatisfied I started working on the jsp engine for [sake](https://github.com/codesake/sake) rubygem.

<!-- more -->

When I'm reviewing the code I use this workflow. Using the commercial tool and
seeing its internal parser output. Then I can focus with manual reviewing
pieces of code that parser _thinks_ they are more prone of being vulnerable.

Manual review involves using ad hoc written script, tools from my toolbox and
eventually something from the opensource world.
In this phase I double check code review findings submitting to a running copy
of the code, some well written pattern attack in order to check if the
suspected vulnerable is either a false positive or not.

## Starting from the view

My tools collection lacks about a good JSP interpreter/parser. Since I'm
writing [codesake.com](http://codesake.com) that it will be an application
security portal, I firedup vim starting a quick and dirty JSP scanning engine
for [sake](https://github.com/codesake/sake).

In a first raw implementation I used regular expression to check:
* the presence of passwords or other suspected sensitive information in the
  code;
* the packages imported by the JSP page. This can be useful to detect third
  party libraries used by the page;
* the values read from the HTTP request
* the variables reflected as output to the user

The approach I used? A lightweight TDD session with irb used instead of rspec.
Remember I want to cook a rough implementation of Sake::Jsp engine. I'll add
rspec when I'll merge it in sake gem hopefully with a better implementation.

## Deep into the scanner

First of all, after the shebang, I turned on UTF-8 encoding since the source
code can have Italian chars inside, I required
[trimmy](https://github.com/thesp0nge/trimmy) a gem I wrote to add a trim
method to ruby String just like PHP and rainbow gem to add a touch of color to
the output.  

``` ruby a rough Sake::Jsp implementation
#!/usr/bin/env ruby
# encoding: UTF-8

require 'trimmy'
require 'rainbow'

# XXX: Performance issue. @lines is looped several times in order to keep tests
# small. A clever approach could be looping the line vector only once instead
# of replicate the loops several times.

```

The above comment is to force myself in finding a clever implementation since
all the small security checks loop in the array full of source code lines, that
it brings the complexity to O(n) where n is the number of lines of code.

Another option is to loop once making all the regular expression and have some
helper methods giving back a particular result.

I added a list of false positive methods that they can be found as output
routine but the value they returned can't be tampered by the user.
The engine behaviour is to use another color to show them, maybe in the future
they can be silently ignored.

The suspected secrets list is very prone to false positives since developers
around the world can save passwords or other secrets in variables called _foo_,
_goofy_, _aaa_, _fooled_ or whatever. This is the main limit of source code
analysis: a tool cannot understand a source code, it can only give a
probabilistic evaluation.
Of course, I don't expect all people agree with this. Which is your opinion?


``` ruby a rough Sake::Jsp implementation
module Sake
  class Jsp

    attr_reader :lines
    attr_reader :filename

    attr_reader :results

    FALSE_POSITIVES = ["request.getContextPath()", "request.getLocalName()", "request.getLocalPort()"]
    SUSPECTED_SECRETS = ["password", "username", "login", "xxx", "todo", "fix", "fixme", "passwd"]

    def initialize(options={})
      @filename = options[:filename] unless options[:filename].nil?
      @lines = {}
      @results = {}

    end
```

I implemented only 2 public methods for Sake::Jsp. 

_sake\_it_ will run all the tests and it is the business logic of the whole
scanning engine and a Sake::Jsp.is_false_positive? that check a variable in the
list of false positive values.

``` ruby a rough Sake::Jsp implementation
    def sake_it
      @lines = read_file
      @results[:reflected]= find_reflected_vars
      @results[:import] = find_imports
      @results[:secrets] = find_secrets
      @results[:entrypoints] = find_attack_entrypoints
    end

    def self.is_false_positive?(var)
      FALSE_POSITIVES.include?(var)
    end

```

{% blockquote %}
The suspected secrets list is very prone to false positives since developers
around the world can save passwords or other secrets in variables called _foo_,
_goofy_, _aaa_, _fooled_ or whatever. This is the main limit of source code
analysis: a tool cannot understand a source code, it can only give a
probabilistic evaluation.
{% endblockquote %}

Here it starts the private implementation of the engine.
_read\_file_ is not that magic. I used the readlines routine to store all the
lines of code in an Array.

``` ruby a rough Sake::Jsp implementation
 private

    def read_file
      return File.readlines(@filename) if File.exists?(@filename)
    end

``` 

_find\_secrets\_ method takes a line of code, it splits it in tokens and then
it loops for every returned word looking in the SUSPECTED_SECRETS Array.
I wont win the Turing Price for this one but eventually it works fair for a
first implementation.

``` ruby a rough Sake::Jsp implementation
    def find_secrets
      ret = []
      @lines.each_with_index do |l, i|
        l = l.unpack("C*").pack("U*")
        l.split.each do |tok|
          ret << {:line=> i+1, :matcher=>tok, :line=>l} if SUSPECTED_SECRETS.include?(l.downcase)
        end
      end
      ret
    end
``` 

A typical activity I do every single time I work on a source code review, is
drawing a mindmap of the interconnection between classes and source code files.
That map helps me in taint propagation and in detecting pieces of code that are
not referenced by anyone so to be marked as unmaintained in the final report.

Finding import declaration from a Jsp page it helps me understand the packages
the page uses and eventually custom Java packages developed by internal team.

``` ruby  a rough Sake::Jsp implementation
    def find_imports
      ret = []
      @lines.each_with_index do |l, i|
        l = l.unpack("C*").pack("U*").trim
        m = /<%@page import="(.*?)"%>/.match(l)
        ret << {:line => i+1, :package=>m[1].trim} unless m.nil?
      end

      ret
    end
``` 

The _find\_imports\_ code can be refined in many ways. The regular expression
here doesn't match a lot of different ways the package import clause can be
written. For sure a single space can lead the import not to be detected.

The _find\_attack\_entrypoints_ it is more interesting. It checks all the
possible read from the request. Remember that all the stuff coming from the
user can be tampered ando so parameters from the HTTP request that they must be
considered as possible attack entrypoints. 

A clever implementation it will consider stuff read from a POST and eventually
other APIs that it can be used to read data in input (File, Database, ...).

``` ruby a rough Sake::Jsp implementation
    def find_attack_entrypoints
      ret = []
      @lines.each_with_index do |l, i|
        l = l.unpack("C*").pack("U*")
        m = /request.getParameter\((.*?)\)/.match(l)
        ret << {:line => i+1, :var => m[1].trim} unless m.nil?

        m = /request.getParameterValues\((.*?)\)/.match(l)
        ret << {:line => i+1, :var => m[1].trim} unless m.nil?

        m = /request.getAttribute\((.*?)\)/.match(l)
        ret << {:line => i+1, :var => m[1].trim} unless m.nil?

      end

      ret
    end

``` 

The _find\_reflected\_vars\_ is the output scanning part of the engine. It
checks all the writing the Jsp page makes as output looking for variables. Of
course it doesn't check for custom written validations, so you have to pay
attention in understanding what this routine gives you as output.

``` ruby a rough Sake::Jsp implementation
    def find_reflected_vars
      ret = []
      @lines.each_with_index do |l, i|
        # <%=avar%> #=> /<%=(\w+)%>/.match(a)[1] = avar
        l = l.unpack("C*").pack("U*")
        m = /<%=(.*?)%>/.match(l)
        ret << {:line => i+1, :var=> m[1].trim} unless m.nil?

        m = /out\.println\((.*?)\)/.match(l)
        ret << {:line => i+1, :var=> m[1].trim} unless m.nil?

        m = /out\.print\((.*?)\)/.match(l)
        ret << {:line => i+1, :var=> m[1].trim} unless m.nil?

        m = /out\.write\((.*?)\)/.match(l)
        ret << {:line => i+1, :var=> m[1].trim} unless m.nil?

        m = /out\.writeln\((.*?)\)/.match(l)
        ret << {:line => i+1, :var=> m[1].trim} unless m.nil?
      end

      ret
    end
``` 

Then the runtime bit it will disappear when I'll merge the engine into sake and codesake.

``` ruby a rough Sake::Jsp implementation

raise "Missing filename." if ARGV[0].nil?

jsp = Sake::Jsp.new({:filename=>ARGV[0]})
jsp.sake_it

jsp.results[:import].each do |res|
  puts "Imported package: #{res[:package]}@#{res[:line]}".color(:white)
end


jsp.results[:reflected].each do |res|
  puts "Reflected variable found: #{res[:var]}@#{res[:line]}".color(:red) unless Sake::Jsp.is_false_positive?(res[:var])
  puts "User should not be able to tamper: #{res[:var]}@#{res[:line]}".color(:yellow) if Sake::Jsp.is_false_positive?(res[:var])
end

jsp.results[:secrets].each do |res|
  puts "Suspected sensitive string found: #{res[:line]}@#{res[:line]}".color(:white)
end

jsp.results[:entrypoints].each do |res|
  puts "Variable read from request: #{res[:var]}@#{res[:line]}".color(:red)
end
``` 

## Off by one

Working on a source code scanning engine is really fun. You will learn a lot
about the target programming language and you learn a lot in terms of hacking
and source code assessment.

From the [failing experience about Owasp Orizon](http://armoredcode.com/blog/untold-owasp-orizon-is-died-and-im-sad-of-it/)
I understood that writing a full featured source code parser is hard and
eventually useless. 

There are so many variables that can influence the way a developer write a
piece of code that trying to understand them is a topic I leave for a sci-fi
fiction like _minority report_.

Source code review is a particular kind of security assessment that involves
creatitivy and that it requires a tool to help security specialist to
understand where he/she should look instead of telling _"I'm pretty sure, you
will find a cross site scripting here"_.

What do you think about this? Have you got different experience about source
code reviews?

**Don't be shy, tell me your story**
