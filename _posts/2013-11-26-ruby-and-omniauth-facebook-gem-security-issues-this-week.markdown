---
layout: post
title: "Ruby and omniauth-facebook gem security issues this week"
date: 2013-11-26 08:39
comments: true
published: true
featured: false
tags: builders ruby omniauth omniauth-facebook csrf heap-overflow vulnerabilities codesake codesake-dawn cve-2013-4164 cve-2013-4562
thumb: source-code.png
level:
hn: 
rd: 
---

A couple of days ago, on Italian Ruby mailing list, 
[Paolo Montrasio](http://www.paolomontrasio.com) reported two security issues
occured in the ruby world. 

Let's see them in detail and add
[codesake-dawn](http://rubygems.org/gems/codesake-dawn) checks for them.

<!-- more -->

## CVE-2013-4164: ruby interpreter heap-based buffer overflow

### The issue

Sometime they are back. In a world where modern programming languages take care
about memory it's uncommon to talk about buffer overflows. 

As described in the [original post](https://www.ruby-lang.org/en/news/2013/11/22/heap-overflow-in-floating-point-parsing-cve-2013-4164/),
a special crafted string when converted to its floating point representation,
it can cause an heap based buffer overflow. 

Buffer overflows can cause the program to stop due to segmentation fault error
(and this can be seen as a denial of service to the target application) or to
arbitrary code execution if the instruction pointer register is successfully
overwritten by the address of the malicious code.

Writing shellcodes is an art. I'm not that good in writing shellcodes but we
will go deep in understanding what a buffer overflow is in future posts.

More importan is that_"...any program that converts input of unknown origin to floating point values (especially common when accepting JSON) are vulnerable."_ 

This means that every code taking output from the external and turning it to a
floating pointer number is vulnerable.

Ruby interpreters affected by this vulnerability are:

* All ruby 1.8 versions
* All ruby 1.9 versions prior to ruby 1.9.3 patchlevel 484
* All ruby 2.0 versions prior to ruby 2.0.0 patchlevel 353
* All ruby 2.1 versions prior to ruby 2.1.0 preview2
* prior to trunk revision 43780

Please note that accordingly to ruby roadmap, there is no further support to
ruby 1.8 since it's marked as end of life. Solution is to upgrade to Ruby 1.9.3
patchlevel 484, ruby 2.0.0 patchlevel 353 or ruby 2.1.0 preview2. If you are
using ruby 1.8 version you **really want** to upgrade it now.

### Fixing with codesake-dawn

I'm a lazy security guy and when I first designed
[codesake-dawn](http://rubygems.org/gems/codesake-dawn) I spent the first two
months coding the underlying security check API. The original idea was that
most of the security checks I want to implement can be described by some very
few general security checks implementing **all** the testing logic. 

A new control in the knowledge base must contain just the parameters the
underlying API needs to say if your code is vulnerable or not.

So, checking for
[CVE-2013-4164](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2013-4164)
means adding in codesake-dawn another ruby class including
[Codesake::Dawn::Kb::RubyVersionCheck](https://github.com/codesake/codesake-dawn/raw/master/lib/codesake/dawn/kb/ruby_version_check.rb) with the information about the ruby
interpreter versions, codesake-dawn has to consider as safe.

``` ruby codesake-dawn CVE-2013-4164 security check
module Codesake
	module Dawn
		module Kb
			# Automatically created with rake on 2013-11-26
			class CVE_2013_4164
				include RubyVersionCheck

        def initialize
          message = "Any time a string is converted to a floating point value, a specially crafted string can cause a heap overflow. This can lead to a denial of service attack via segmentation faults and possibly arbitrary code execution. Any program that converts input of unknown origin to floating point values (especially common when accepting JSON) are vulnerable."

          super({
            :name=>"CVE-2013-4164",
            :cvss=>"not assigned",
            :release_date => Date.new(2013, 11, 23),
            :cwe=>"",
            :owasp=>"A9", 
            :applies=>["rails", "sinatra", "padrino"],
            :kind=>Codesake::Dawn::KnowledgeBase::RUBY_VERSION_CHECK,
            :message=>message,
            :mitigation=>"All users are recommended to upgrade to Ruby 1.9.3 patchlevel 484, ruby 2.0.0 patchlevel 353 or ruby 2.1.0 preview2.",
            :aux_links=>["https://www.ruby-lang.org/en/news/2013/11/22/heap-overflow-in-floating-point-parsing-cve-2013-4164/"]
          })

          self.safe_rubies = [{:engine=>"ruby", :version=>"1.9.3", :patchlevel=>"p484"}, {:engine=>"ruby", :version=>"2.0.0", :patchlevel=>"p353"}, 
            {:engine=>"ruby", :version=>"2.1.0", :patchlevel=>"preview2"}]

				end

			end
		end
	end
end
```

## CVE-2013-4562: the omniauth-facebook gem cross site request forgery

### The issue

A couple of weeks ago, on [OSS Security mailing list](http://www.openwall.com/lists/oss-security) it has been
[reported](http://www.openwall.com/lists/oss-security/2013/11/12/5) that the
[omniauth-facebook](http://rubygems.org/gems/omniauth-facebook) gem version 1.4.1 is affected by a 
[cross site request forgery](https://www.owasp.org/index.php/Top_10_2013-A8-Cross-Site_Request_Forgery_\(CSRF\))
vulnerability.

omniauth-facebook ruby gem introduced in the vulnerable version an anti-CSRF
mechanism that it's broken. Since the gem supports setting a per-request state
parameter by storing it in the session, it is possible to circumvent the
automatic CSRF protection. 

Versions prior 1.4.1 are not vulnerable since they don't implement such kind of
security mechanism, so you **must not** downgrade your gem in order to fix this
vulnerability. The only way to mitigarte the vuln is to upgrade the gem to
version 1.5.0.

A cross site request forgery occurs when an attacker is able to inject
arbitrary javascript code in your users' browser (in example using a cross site
scripting) with the intent to perform automated requests to a target (usually a
third party website) with the victim credentials. The offended website saw
requests coming from your users' browser and eventually it would trust them
since they are also authenticated on the target site (that it is the real
attack victim, your vulnerable website is used attacking endpoint)). 

### Fixing with codesake-dawn

Implementing a check against a particular version of a ruby gem with codesake
dawn is easy. You must create a new class, implementing the checks and
including the [Codesake::Dawn::Kb::DependencyCheck](https://github.com/codesake/codesake-dawn/raw/master/lib/codesake/dawn/kb/dependency_check.rb) class.

``` ruby codesake-dawn CVE-2013-4562 security check
module Codesake
	module Dawn
		module Kb
			# Automatically created with rake on 2013-11-26
			class CVE_2013_4562
				include DependencyCheck

				def initialize
          message = "Because of the way that omniauth-facebook supports setting a per-request state parameter by storing it in the session, it is possible to circumvent the automatic CSRF protection. Therefore the CSRF added in 1.4.1 should be considered broken. If you are currently providing a custom state, you will need to store and retrieve this yourself (for example, by using the session store) to use 1.5.0."
          super({
            :name=>"CVE-2013-4562",
            :cvss=>"not assigned",
            :release_date => Date.new(2013, 11, 14),
            :cwe=>"",
            :owasp=>"A9", 
            :applies=>["rails", "sinatra", "padrino"],
            :kind=>Codesake::Dawn::KnowledgeBase::DEPENDENCY_CHECK,
            :message=>message,
            :mitigation=>"You must upgrade at least to 1.5.0 or later",
            :aux_links=>["https://groups.google.com/forum/#!msg/ruby-security-ann/-tJHNlTiPh4/9SJxdEWLIawJ"]
          })

          self.safe_dependencies = [{:name=>"omniauth-facebook", :version=>['1.5.0']}]

				end
			end
		end
	end
end
```

## Enjoy it

All the changes to codesake-dawn are already online with [this commit](https://github.com/codesake/codesake-dawn/commit/d7441b1c4079a69950f0ed2f6104b979bbac3d2d).

**BONUS TRACK**

While preparing the post I noticed I missed also a [cocaine](http://rubygems.org/gems/cocaine) ruby gem
security announce. The Cocaine gem 0.4.0 through 0.5.2 for Ruby allows
context-dependent attackers to execute arbitrary commands via a crafted has
object, related to recursive variable interpolation. 

The CVE it was assiegned is [CVE-2013-4457](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2013-4457) and the implementation in codesake-dawn is done this way:

``` ruby codesake-dawn CVE-2013-4457 security check
module Codesake
	module Dawn
		module Kb
			# Automatically created with rake on 2013-11-26
			class CVE_2013_4457
				include DependencyCheck

				def initialize
          message="The Cocaine gem 0.4.0 through 0.5.2 for Ruby allows context-dependent attackers to execute arbitrary commands via a crafted has object, related to recursive variable interpolation."
          super({
            :name=>"CVE-2013-4457",
            :cvss=>"not assigned",
            :release_date => Date.new(2013, 10, 22),
            :cwe=>"",
            :owasp=>"A9", 
            :applies=>["rails", "sinatra", "padrino"],
            :kind=>Codesake::Dawn::KnowledgeBase::DEPENDENCY_CHECK,
            :message=>message,
            :mitigation=>"You must upgrade to cocain gem version 0.5.3 or later",
            :aux_links=>["https://groups.google.com/forum/#!topic/ruby-security-ann/3XTGFbAJoTg"]
          })

          self.safe_dependencies = [{:name=>"cocaine", :version=>['0.5.3', '0.4.9999']}]


				end
			end
		end
	end
end
``` 

Enjoy it!


