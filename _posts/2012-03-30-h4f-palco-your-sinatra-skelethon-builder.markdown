---
layout: post
title: "H4F - palco: your Sinatra skeleton builder"
date: 2012-03-30 9:00
comments: true
published: true
tags: h4f ruby rubygem hacking builders source git sinatra webapp
hn: http://news.ycombinator.com/item?id=3774901
rd: http://www.reddit.com/r/ruby/comments/rknc8/h4f_palco_your_sinatra_skeleton_builder/
---

[Sinatra](http://www.sinatrarb.org) is a powerful and easy to use ruby based
DSL to create web applications and powerful APIs.

In this first [Hack for fun](http://armorecode.com/blog/categories/h4f)
category post I will show you a simple real life [rubygem](http://rubygems.org)
project implemented in a couple of afternoons while I'm rerecovering from a
surgery.

Palco: a simple way to create sinatra based directory layouts.

<!-- more -->

You know, I'm lazy and I don't have too much memory, so starting everytime from
scratch with a new Sinatra project is something so boring.

So, why don't automate the process? I'm also a ruby hacker so why don't write a
simple gem acting as template generator?

That's why I started [palco](https://github.com/thesp0nge/palco), that is the
Italian word for stage, but let's cross all the project steps.

## Finding a killing name

Trust me, you will work with more enthusiasm if you find a killing name for
your project. What about if I chose _sinatradircreator_ or _sinatragenerator_ ? 

I bet I would stop coding after 5 minutes.

Is palco a killing name? Well, it's Italian so most of people may find it
amusing. Definitely it's on music, Frank Sinatra world related. And it's
finally short and easy to pronunce.

I like it.

## Creates a skeleton and feed a repo

As described in [this blog post](http://rakeroutes.com/blog/lets-write-a-gem-part-one/) 
I'll use [bundler](http://gembundler.com/) to manage the whole process.

First of all, I'll create the gemset. I'm going to use ruby version 1.9.2 but
this should not affect your fork if you want to use a different interpreter
version.

```
rvm --create use 1.9.2@palco
```

Next, I'll create the skeleton using previously installed bundler tool.

``` 
bundle gem palco
``` 

Bundle also initialize a git repository for you. So you may choose the git
service you like most (self hosted, [gitorious](http://gitorious.org/) or
[github](https://github.com). I love the latter so I setup a repository for
[palco](https://github.com/thesp0nge/palco) and saved the remote address so I
can push my commits into it.

```
git remote add origin git@github.com:thesp0nge/palco.git
git commit -m 'first import'
git push -u origin master
```

The repository is in place, ready to receive our commits.

## Documenting the code

Documentation is most than important for a project. That's why I started two
different README files for palco (I never do this, but I do think it's
important... and since the effort is risible, I guess I'll start writing more
than a single README for a project):

* a [main](https://github.com/thesp0nge/palco/blob/master/README.md) README file 
* a README [file](https://github.com/thesp0nge/palco/blob/master/lib/README.md) for API

In the source code, I used [Tom Werner](http://tomdoc.org/) documentation format.

## Some toughts on architecture

Since I want to be able to create directories and files that are different from
a Sinatra application and a Sinatra extension, it's clear that I must move all
the business logic in a core class.

This core class must be generic enough to take the list of items from being
created and than with a generate method it must be able to do all the dirty
work. 

What I need is a [Palco::Base](https://github.com/thesp0nge/palco/blob/master/lib/palco/base.rb) 
class.

## The tests are the pathway

[Test driven](http://en.wikipedia.org/wiki/Test-driven_development) and
[behavior driven](http://en.wikipedia.org/wiki/Behavior_Driven_Development)
development strategies repeat this as a mantra: start writing test cases and
expectation, you must define first of all what's the expected output and then
you start implementing to reach your _desiderata_.

So I started mocking up Palco::Base after I had defined some test using
[rspec](http://rspec.info/).

### Don't overmelt your rspec

This is the Palco::Base implementation at the time I took an hard lesson.

{% gist 2205758 palco.rb %}

First hard lesson. If your test cases is too dense, it won't work and you don't
know why. You must keep also the test case as simple as possible.

Look at this:

{% gist 2205758 palco_base_helper.rb %}

The skeleton project was created but testing code went into a loop... there is
a bug in the Palco::Base and you have 5 minutes starting from now to spot it.

Do you?

### Kytss - keep your tests simple, stupid

Look at this method:

```
def generate?
  return self.generate
end 
``` 

Do you notice something strange? Well, this method was designed to answer the
question _did I called the generate method?_ using the class attribute
_generated_.

But, for a typo I return the generate method, not the generated attribute and
so, the bug is served.

So I break the rspec in smaller tests in the test case
[file](https://github.com/thesp0nge/palco/blob/master/spec/palco_base_spec.rb)
that is online.

## Get ready to deploy

Using the close test and then code loop, I solved the extension skeleton
creation with a binary script ready to be distributed as rubygem.

Before hitting the Internet, some touches to the gemspec must be done. Make
sure to fill the summary and the description so people can have a tip on what
your gem does and if it can be useful to them. 

``` ruby palco.gemspec

# -*- encoding: utf-8 -*-
$:.push File.expand_path("../lib", __FILE__)
require "palco/version"

Gem::Specification.new do |s|
  s.name        = "palco"
  s.version     = Palco::VERSION
  s.authors     = ["Paolo Perego"]
  s.email       = ["thesp0nge@gmail.com"]
  s.homepage    = "http://armoredcode.com"
  s.summary     = %q{Creates Sinatra application and extension directory layout}
  s.description = %q{Creates Sinatra application and extension directory layout}

  s.rubyforge_project = "palco"

  s.files         = `git ls-files`.split("\n")
  s.test_files    = `git ls-files -- {test,spec,features}/*`.split("\n")
  s.executables   = `git ls-files -- bin/*`.split("\n").map{ |f| File.basename(f) }
  s.require_paths = ["lib"]

  # specify any dependencies here; for example:
  # s.add_development_dependency "rspec"
  s.add_runtime_dependency "rainwbow"
end
```

Tune the version to make sure that you set it to 1.0.0 and then go ahead and
rule the world.

## Launch

I added some code to autofill some files with stub content. I choose New BSD
license for LICENSE file and the basic code for version.rb or extension code
and class code initialization is pretty standard so palco can help in init the
work for the developer.

Launching the project is easy. Rakefile has an install task that pushes the gem
in [rubygems.org](http://rubygems.org) website.

Get excited? Do you want to try [palco](https://rubygems.org/gems/palco)?

```
gem install palco
``` 

Have fun!

## Isn't this post completely off topic, is it?

Well, true to be told building a tool to automate some tasks and that it is
useful to your work is a common hackish way of life and it's fun. So "Hack for
fun" series will be about building tools. Next post will be _"Understanding
your risk - Error messages"_ and it will about information an attacker can gain
when applications don't care enough about information disclosed in error or
status messages.

