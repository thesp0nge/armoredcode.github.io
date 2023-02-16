---
layout: post
title: "codesake engine and two weeks of BDD development"
date: 2012-12-23 09:00
comments: true
published: true
featured: false
tags: builders jsp ruby sake code-review sast codesake application-security appsec
thumb: sake.png
level:
hn:
rd:
---

Two weeks ago, I [posted an article](http://armoredcode.com/blog/driven-by-real-world-task-code-reviewing-jsp-using-regular-expression/)
about a real world source code security review. Using regular expressions I was
able to spot interesting things over JSP files I was reviewing.
Client was happy.
My workflow was smooth.

And codesake engine has a great part of it.

<!-- more -->

**Disclaimer:**_Some of the links contained within this post have my Amazon referral
ID. This provides me with a small commission for each sale.  Thank you for your
support._

After [the failing Owasp Orizon experience](http://armoredcode.com/blog/untold-owasp-orizon-is-died-and-im-sad-of-it/)
I have a strong belief. A source code static analysis tool **has not to do your
work**, instead it has to help an application security specialist in looking at
the right place.

Building a full featured compiler in order to build an abstract syntax tree,
inspecting the code in deep, making call backtracing and assertion over the
call flow is a great programming task. My opinion on this topic is that people
designing compilers are _artists_.

My thought is that you can't fully automate developers' work without
[falling in a pitfall](http://armoredcode.com/blog/the-hidden-pitfalls-in-automatic-source-code-review/)
and it would be more useful having something that helps a security specialist
to look in the right place. It will be a human to say if in that place a
security issue was found.

## 2 weeks of BDD workflow

Two weeks ago I started with the
[codesake](https://github.com/codesake/codesake) gem that hopefully it will be
also part of the application security SaaS I will launch in 2013.

The idea is simple. As an appsec guy I know which are the coding patterns that
they can introduce vulnerabilities.

The tool will do the _dirty grep_ work for you and it will be up to you to mark
a suspicious unsafe pattern as a real appsec vulnerabilities.

I used the book ["Build Awesome Command-Line Applications in Ruby"](http://www.amazon.com/exec/obidos/ASIN/1934356913/armoredcode-20/ref=no-sim/),
by David Bryant Copeland as reference for creating a robust command line tool.
I purchased this book in July and finished almost in a single deep reading
evening. It will deserve a full review one of these days.

In the "Test, Test, Test" chapter I discovered aruba rubygem that adds some
interesting features to cucumber dedicated in testing command line interfaces.

In 2 weeks of work during early morning commuting I wrote code thats:

* scan plain text file for _reserved_ words. That means looking for keywords
  like _password_ or _fixme_ or _todo_, in order to check if text file has some
  interesting information or it documents some feature that has known bugs. As
  attacker, if you find a comment like _todo: must fix this error checking
  routine_ you may want to start evaluating how to override that buggy code.
* scan jsp files for reserved keywords, variables that are written in output,
  reads from HTTP requests, cookies.

That it's almost the same stuff Owasp Orizon was checking but with a more
clumbered internal architecture.

codesake has engines dedicated to a particular file kind, with internal _adhoc_
checks. They share a common interface that is the analyse method returning an
array. Eventually this array contains String objects that the binary script
will put on standard output.
Every engine will be responsible of formatting the output.

A kernel was in place to act like multiplexer for choosing the correct engine:

``` ruby Codesake::Kernel
require 'singleton'

module Codesake
  class Kernel
    include Singleton

    attr_reader   :engine

    NONE    = 0
    TEXT    = 1
    JSP     = 2
    UNKNOWN = -1

    def choose_engine(filename, options)


      engine = nil

      case detect(filename)
      when TEXT
        engine = Codesake::Engine::Text.new(filename)
      when NONE
        engine = Codesake::Engine::Generic.new(filename)
      when JSP
        engine = Codesake::Engine::Jsp.new(filename, options)
      end
      engine
    end

    def detect(filename)
      return NONE if filename.nil? or filename.empty?
      return TEXT if Codesake::Engine::Text.is_txt?(filename)
      return JSP if (File.extname(filename) == ".jsp")


      return UNKNOWN
    end
  end
end
```

The kernel is called by the binary script that asks it to choose the correct engine for a given target.

``` ruby bin/codesake
...

cli = Codesake::Cli.new
kernel = Codesake::Kernel.instance

options=cli.parse(ARGV)

...

cli.targets.each do |target|
  puts "processing #{target[:target]}" if target[:valid]
  $stderr.puts "can't find #{target[:target]}".color(:red) if ! target[:valid]

  engine = kernel.choose_engine(target[:target], options)

...

  results = engine.analyse
  results.each do |res|
    $stdout.puts "#{res}"
  end
end

```

### Working on a new feature

Let's say we want to introduce the scan for cookie security check in jsp
engine. The need is that we want, as application security specialist, to check
if cookies are used in a Jsp page in order to check if we can tamper it or even
looking for secrets and sensitive informations.

We start in writing the rspec code for the functionality we want to add in
order to make unit testing.

``` ruby spec/jsp_engine_spec.rb
...

describe Codesake::Engine::Jsp do
  before(:all) do
    File.open("test.jsp", "w") do |f|
      f.write(jsp_content)
    end
    @jsp = Codesake::Engine::Jsp.new("test.jsp", {})
    @jsp.analyse
  end

  after(:all) do
    File.delete("test.jsp") if File.exists?("test.jsp")
  end

  it_behaves_like Codesake::Utils::Files
  it_behaves_like Codesake::Engine::Core

  ...

  it "analyses a jsp file for cookies" do
   expected_result = [{:line=>51, :name=>"name", :value=>"a_value", :var=>"c"}, {:line=>52, :name=>"second", :value=>"12", :var=>"cc"}]
   @jsp.cookies.should == expected_result
 end
end
```

Tests for Codesake::Engine::Jsp include some common ground routines for file
handling and text grabbing that are included in external modules and that are
checked with the it_behaves_like rspec clause.
Checks for modules are so integrated in this rspec run for my Jsp engine.

We expect a _cookies_ array contains hashes from every cookie read in a test
jsp file that is in the spec file (look in the sources, I won't include it here
for simplicity).

Running the unit tests we had this feature to fail as espected. Good. Let's
move on and write our cucumber scenario for the integration test before we
proceed.

``` ruby features/codesake_process_jsp_file.feature
Feature: codesake process a jsp page
  When a Jsp file is given as input, codesake analyses it with the
  Codesake::Engine::Jsp engine for security issues.

  When a Jsp file is analyzed the following information will be gathered:

  * imported packages
  * variable read from requests
  * cookies created
  * reserved keywords

  ...

  Scenario: codesake processing the file finds cookies that are created by the page
    Given the jsp file "/tmp/existing.jsp" with cookies does exist
    When I successfully run `bundle exec codesake /tmp/existing.jsp`
    Then the stdout should contain "cookie \"name\" found with value: \"a_value\" (/tmp/existing.jsp@51)"
    And the stdout should contain "cookie \"second\" found with value: \"12\" (/tmp/existing.jsp@52)"
```

The When clause and the Then matchers are all managed by aruba gem. Pretty neat.
Running also integration tests all act as expected. Our binary doesn't know
anything about cookie check output. So all red here.

We can start implementing our feature.

First we add a cookies attribute to the Codesake::Engine::Jsp class, so our
unit test rspec file can stop complaining about the cookie attribute that is
missing.
Now rspec complains that cookies has nil value instead of the expected result.

``` ruby Codesake::Engine::Jsp
module Codesake
  module Engine
    class Jsp
      include Codesake::Utils::Files
      include Codesake::Utils::Secrets
      include Codesake::Engine::Core

      ...

      attr_reader :cookies
      ...
```

To solve this turning the spec green, we have to implement the find\_cookies
routine matching with regular expressions the Java statements creating a
Cookie.

> This show also how this approach can fail. If the regular expression is poorly
> written codesake won't be able to detect pattern and then you may miss some
> findings.


``` ruby Codesake::Engine::Jsp.find_cookies
def find_cookies
  ret = []

  @file_content.each_with_index do |l, i|
    l = l.unpack("C*").pack("U*")
    m = /Cookie (.*?) = new Cookie \("(.*?)",(.*?)\)/.match(l);
    ret << {:line => i+1, :var => m[1].trim, :name => m[2].trim.gsub("\"", ""), :value => m[3].trim.gsub("\"", "")} unless m.nil?

    m = /Cookie (.*?) = new Cookie\("(.*?)",(.*?)\)/.match(l);
    ret << {:line => i+1, :var => m[1].trim, :name => m[2].trim.gsub("\"", ""), :value => m[3].trim.gsub("\"", "")} unless m.nil?

    m = /(.*?) = new Cookie \("(.*?)",(.*?)\)/.match(l);
    ret << {:line => i+1, :var => m[1].trim, :name => m[2].trim.gsub("\"", ""), :value => m[3].trim.gsub("\"", "")} unless m.nil?

  end
  ret
end
```

Assign find\_cookies values to cookies attribute will make our test to succeed.
A regex improvement that it is underway is to skip whitespaces in some Java
statement.


Running integration tests, cucumber will complain about the missing output for the binary tool.
As I said before, it's up to the engine to prepare output for binary script; I
make this choice to avoid the main ruby code to deal with different results
format, instead of just having a string to show.

Let's add cookie reporting code:

``` ruby Codesake::Engine::Jsp.analyse
def analyse
  ret =  []
  ...

  @cookies            = find_cookies

  ...
    @cookies.each do |c|
      ret << "cookie \"#{c[:name]}\" found with value: \"#{c[:value]}\" (#{@filename}@#{c[:line]})"
    end


    ret
  end
```

Now everything is green. We implemented our cookies scan functionality, if a
JSP file will create a Cookie for us we can detect it and see if we have to be
worried about it or not.

### Mixing up static and dynamic

But I don't want to implement a grep++ in ruby. The future is hybrid source
code analysis and codesake (and [codesake.com](http://codesake.com) as well),
will move further in this direction.

Soon, one or more dynamic actions will be associated to the static finding. In
example, if a reflected XSS has been found in a JSP page it will be a clever
approach to close test this finding making a call to that url tampering the
parameter we marked as suspected.

### Roadmap

Hopefully I will launch [codesake.com](http://codesake.com) as SaaS application
security platform before summer 2013, so codesake engine that it will be one of
the core engines it will reach its first major release.

codesake rubygem will always remain an opensource, MIT licensed, sast engine.
Of course I'll add more advanced functionalities in the commercial part of the
codesake.com portal, mostly designed for ruby powered web applications
(Sinatra, Padrino and Rails).

Some nice to have features:

* support for JAVA, Ruby and PHP
* different output formats, including SQL
* improve regular expressions
* much more better documentation

Enjoy it!
