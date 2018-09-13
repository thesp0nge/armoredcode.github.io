---
layout: post
title: "Codesake Dawn: the new security source code scanner for ruby"
date: 2013-05-29 08:24
comments: true
published: true
featured: true
categories: ruby appsec code-review codesake codesake-dawn security sast
thumb: dawn.png
level:
hn: 
rd: 
---

## Prologue
It was a dark and stormy night back in 2006 when I started the 
[Owasp Orizon project](http://www.owasp.org/index.php/GPC_Project_Details/OWASP_Orizon_Project)
which I dedicated an _ad hoc_ story on this blog back in 
[november 2012](http://armoredcode.com/untold-owasp-orizon-is-died-and-im-sad-of-it/)

In an [October post](http://armoredcode.com/the-hidden-pitfalls-in-automatic-source-code-review/)
I talked about what I dislike on commercial statical analysis tools over there.

[This winter](http://armoredcode.com/codesake-engine-and-two-weeks-of-bdd-development/)
I was thinking about bootstrapping [an application security
startup](http://codesake.com) giving the world hybrid web application security
testing services. By hybrid testing I mean (and a lot of venerable security
guys around the world tool) that over a target both static analysis than
penetration testing activities has to be performed in order to achieve a good
security level.

I **do** believe I'll bootstrap [codesake.com](http://codesake.com) but I
narrowed my goal a little bit in order to be more focused and
[Codesake::Dawn](https://github.com/codesake/codesake_dawn) is one of the goal
I'm working hard to achieve soon.

<!-- more -->

## Avoid the pitfalls
[Codesake::Dawn is a gem](https://rubygems.org/gems/codesake-dawn) providing a
security source code analyzer for web applications written in
[ruby](http://ruby-lang.org/en). First goal is being focused on a **single**
programming language and supporting it **good**.

I do hate tools with fancy *gonna catch'em all* commercials. KISS pricinple is
still valid, isn't it?!? 

No matter which is the framework you use to build your next level web
application, if you use ruby, you really want to install dawn and use it to
spot security findings.
Second goal: being **framework** independent.

Third goal: **rely on the source code**. This is easy with ruby that is an
interpreted language, so compile it for a static analysis is a non sense. The
idea is that our target is something the developer wrote not something a
compiler produced.

## Codesake::Dawn under the cover

### The knowledge base
The core of the tools is of course the [knowledge base](https://raw.github.com/codesake/codesake_dawn/master/lib/codesake/dawn/knowledge_base.rb).

Codesake::Dawn has (in the 0.60 version) three different kind of security checks:

* pattern matching
* dependency check
* ruby interpreter version

Each security check is a separate ruby class, so knowledge base has to include
them all in its very beginning. Each security check can be also be applied only
to one mvc from rails, sinatra, padrino or it can be applied to more than a
single mvc at the time.

The core part of the knowledge base class is the load\_security\_checks method
called during class initialization. It creates an instance of each security
check class contained in the knowledge base, populating an array of ruby
objects to be used during the analysis.

``` ruby Codeasake::Dawn::Knowledge.base.load_security_checks
def self.load_security_checks
  [  
    Codesake::Dawn::Kb::NotRevisedCode.new,
    Codesake::Dawn::Kb::CVE_2011_2931.new, 
    Codesake::Dawn::Kb::CVE_2012_2660.new, 
    Codesake::Dawn::Kb::CVE_2012_2661.new, 
    Codesake::Dawn::Kb::CVE_2012_2694.new, 
    Codesake::Dawn::Kb::CVE_2012_2695.new, 
    Codesake::Dawn::Kb::CVE_2012_3465.new, 
    Codesake::Dawn::Kb::CVE_2012_6496.new, 
    Codesake::Dawn::Kb::CVE_2012_6497.new,
    Codesake::Dawn::Kb::CVE_2013_0155.new,
    Codesake::Dawn::Kb::CVE_2013_0156.new,
    Codesake::Dawn::Kb::CVE_2013_0175.new,
    Codesake::Dawn::Kb::CVE_2013_0233.new,
    Codesake::Dawn::Kb::CVE_2013_0269.new,
    Codesake::Dawn::Kb::CVE_2013_0276.new,
    Codesake::Dawn::Kb::CVE_2013_0277.new,
    Codesake::Dawn::Kb::CVE_2013_0284.new,
    Codesake::Dawn::Kb::CVE_2013_0285.new,
    Codesake::Dawn::Kb::CVE_2013_0333.new,
    Codesake::Dawn::Kb::CVE_2013_1655.new,
    Codesake::Dawn::Kb::CVE_2013_1656.new,
    Codesake::Dawn::Kb::CVE_2013_1800.new,
    Codesake::Dawn::Kb::CVE_2013_1801.new,
    Codesake::Dawn::Kb::CVE_2013_1802.new,
    Codesake::Dawn::Kb::CVE_2013_1821.new,
    Codesake::Dawn::Kb::CVE_2013_1854.new, 
    Codesake::Dawn::Kb::CVE_2013_1855.new, 
    Codesake::Dawn::Kb::CVE_2013_1856.new, 
    Codesake::Dawn::Kb::CVE_2013_1857.new, 
    Codesake::Dawn::Kb::CVE_2013_1875.new, 
    Codesake::Dawn::Kb::CVE_2013_1898.new, 
    Codesake::Dawn::Kb::CVE_2013_1911.new, 
    Codesake::Dawn::Kb::CVE_2013_1933.new, 
    Codesake::Dawn::Kb::CVE_2013_1947.new, 
    Codesake::Dawn::Kb::CVE_2013_1948.new, 
    Codesake::Dawn::Kb::CVE_2013_2615.new, 
    Codesake::Dawn::Kb::CVE_2013_2616.new, 
    Codesake::Dawn::Kb::CVE_2013_2617.new, 
    Codesake::Dawn::Kb::CVE_2013_3221.new, 
  ]
end
```

An helper method can retrieve security checks that can be applied to a
particular mvc framework:

``` ruby Codesake::Dawn::KnowledgeBase all_by_mvc method
def all_by_mvc(mvc)
  ret = []
  @security_checks.each do |sc|
    ret << sc if sc.applies_to?(mvc)
  end

end
```

And there are also helpers to make the search more intuitive for who is writing a security check.

``` ruby some helpers to search the knowledge base given a particular mvc
def all
  @security_checks
end

def all_sinatra_checks
  self.all_by_mvc(:sinatra)
end

def all_rails_checks
  self.all_by_mvc(:rails)
end

def all_padrino_checks
  self.all_by_mvc(:padrino)
end

def all_rack_checks
  self.all_by_mvc(:rack)
end
```

### The engine

Security checks are loaded when the [Codesake::Dawn::Engine](https://raw.github.com/codesake/codesake_dawn/master/lib/codesake/dawn/engine.rb)
 class is created.  Engine class is a singleton dawn uses to match checks and
other information against its target, there are three kind of engines one for
each mvc framework supported by dawn:

* [Codesake::Dawn::Rails](https://raw.github.com/codesake/codesake_dawn/master/lib/codesake/dawn/rails.rb)
* [Codesake::Dawn::Sinatra](https://raw.github.com/codesake/codesake_dawn/master/lib/codesake/dawn/sinatra.rb
* Codesake::Dawn::Padrino that it will be added soon when padrino framework will be officially supported

When an engine is created it loads the knowledge base security checks that are
applicable to it
``` ruby Codesake::Dawn::Engine load_knowledge_base method
def load_knowledge_base
  @checks = Codesake::Dawn::KnowledgeBase.new.all_by_mvc(self.name)
  @checks
end
```

The core part of the engine class is the *apply* family methods. An engine can
apply a single security check or all the security checks in the knowledge base.

Here we will show the apply\_all method.

``` ruby Codesake::Dawn::Engine apply_all method
def apply_all
  load_knowledge_base if @checks.nil?
  return false if @checks.empty?

  @checks.each do |check|
    @applied << { :name => name }
    check.ruby_version = self.ruby_version[:version]
    check.detected_ruby  = self.ruby_version if check.kind == Codesake::Dawn::KnowledgeBase::RUBY_VERSION_CHECK
    check.dependencies = self.connected_gems if check.kind == Codesake::Dawn::KnowledgeBase::DEPENDENCY_CHECK
    check.root_dir = self.target if check.kind  == Codesake::Dawn::KnowledgeBase::PATTERN_MATCH_CHECK
    @vulnerabilities  << {:name=> check.name, :message=>check.message, :remediation=>check.remediation , :evidences=>check.evidences} if check.vuln?
    @mitigated_issues << {:name=> check.name, :message=>check.message, :remediation=>check.remediation, :evidences=>check.evidences} if check.mitigated?
  end

  true

end
```

It's up to this method to pass specific project information to the security
check to see if the target application is vulnerable or not.

Security analysis results are stored in two different arrays:

* vulnerabilities
* mitigated_issues

The *vulnerabilities* array contains security checks that affect target application. The *mitigated_issues* contains potentially dangerous security issues that however are mitigated for a particular side condition like the version of the ruby interpreter or a particular gem in the Gemfile.

As example you may want to take the
[CVE-2013-0333](https://raw.github.com/codesake/codesake_dawn/master/lib/codesake/dawn/kb/cve_2013_0333.rb)
security check. There are some versions of the rails framework that are
affected by a SQL injection because JSON data are not properly transformed into
YAML. However this vulnerability is mitigated if the project uses the yajl
rubygem. 

So a typical dawn security analysis work out is:

* create an engine, setting up the target
* loading the knowledge base
* applying all the security checks
* loop through the vulnerabilities found and create the output

### The checks

The skelethon of the security part is the
[Codesake::Dawn::Kb::BasicCheck](https://raw.github.com/codesake/codesake_dawn/master/lib/codesake/dawn/kb/basic_check.rb)
that is the basis for the three different kind of security checks out there.

All the core security checks modules include BasicCheck and they expose a
boolean method vuln? that says if the target is vulnerable or not a particular
knowledge base check. Those modules are:

* [Codesake::Dawn::Kb::PatternMatchCheck](https://raw.github.com/codesake/codesake_dawn/master/lib/codesake/dawn/kb/pattern_match_check.rb)
* [Codesake::Dawn::Kb::DependencyCheck](https://raw.github.com/codesake/codesake_dawn/master/lib/codesake/dawn/kb/dependency_check.rb)
* [Codesake::Dawn::Kb::RubyVersionCheck](https://raw.github.com/codesake/codesake_dawn/master/lib/codesake/dawn/kb/ruby_version_check.rb)

A generic check in the knowledge base includes one of those modules, populating
the super call during initialization and no more. All the logic is in those
three core security check module.

As example, let's talk again about CVE-2013-0333 security check. This is the way it looks like.

``` ruby Codesake::Dawn::Kb::CVE-2013-0333
module Codesake
	module Dawn
		module Kb
			# Automatically created with rake on 2013-04-30
			class CVE_2013_0333
				include DependencyCheck

				def initialize
          message = "lib/active_support/json/backends/yaml.rb in Ruby on Rails 2.3.x before 2.3.16 and 3.0.x before 3.0.20 does not properly convert JSON data to YAML data for processing by a YAML parser, which allows remote attackers to execute arbitrary code, conduct SQL injection attacks, or bypass authentication via crafted data that triggers unsafe decoding, a different vulnerability than CVE-2013-0156." 
          super({
            :name=>"CVE-2013-0333",
            :cvss=>"AV:N/AC:L/Au:N/C:P/I:P/A:P",
            :release_date => Date.new(2013, 1, 30),
            :cwe=>"",
            :owasp=>"A9", 
            :applies=>["rails"],
            :kind=>Codesake::Dawn::KnowledgeBase::DEPENDENCY_CHECK,
            :message=>message,
            :mitigation=>"Please upgrade rails version at least to 2.3.16 or 3.0.20. As a general rule, using the latest stable rails version is recommended.",
            :aux_links=>["https://groups.google.com/forum/?fromgroups=#!topic/rubyonrails-security/1h2DR63ViGo"]
          })

          self.safe_dependencies = [{:name=>"rails", :version=>['2.3.16', '3.0.20']}]
          self.aux_mitigation_gem = {:name=>"yajl", :versione=>['any']}
				end
			end
		end
	end
end
```

## Off by one

The [competitive matrix](https://github.com/codesake/codesake_dawn/blob/master/Competitive_matrix.md)
out there is intended to make a comparison between dawn and other ruby security
source code scanners out there. Feel free to help me in filling the table.

dawn companion will be a future gem named codesake-dusk containing the dynamic testing part.

Soon, when dawn version 1.0 will be out it will be also served as paid API
served in [codesake.com](http://codesake.com) application security startup but
this is another story I'll talk about in the future.

Enjoy it!


_image courtesy by [John](http://www.flickr.com/photos/shebalso/)_
