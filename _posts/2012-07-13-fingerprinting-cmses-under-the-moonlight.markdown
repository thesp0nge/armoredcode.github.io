---
layout: post
title: "Fingerprinting CMSes under the moonlight"
date: 2012-07-16 08:46
comments: true
published: true
featured: false
tags: builders breakers ruby hacking gems cms information-gathering pentest fingerprint pentest-with-ruby
hn: http://news.ycombinator.com/item?id=4249675
rd: http://redd.it/wmt5l
thumb: gengis.png
---

Yesterday I was surfing the web for inspiration for redesign
[armoredcode.com](http://armoredcode.com) layout and I was digging in some
webdesign template websites.

A lot of CMSes use the _generator_ meta tag for statistic purposes. Wouldn't
it be fun having something automated telling server operating system and CMSes
version with single HTTP get?

Yesterday, after dinner I relaxed my self and I started
[gengiscan](https://github.com/thesp0nge/gengiscan).

<!-- more -->

## Scenario

Imagine you're in the early stage of your security assessment over a web
application. You've got your target and you want to know more about it.

You're not one of that testers firing up an _point and click automated tool_
saying you're a security expert. You want to attack the site like a pro, and
**of course** you're authorized to make activities.

![]({{site.url}}/images/kaboom.gif)

Let's make this assumption now, your target is a CMS powered website, so you're
interested in stuff like privilege escalation over content management backend,
something basic like cross site scripting but the real deal would be finding
[SQL injections](http://www.thehostingnews.com/sql-injection-hits-yahoo-voices-453k-logins-leaked-25021.html)
and get into the backend database.

So, your first problem is to understand which kind of backend software is
serving your target. For statistical purposes, CMSes often expose the software
used in the backend in a _meta_ html tag called "generator".

> The real deal would be finding SQL injections and get into the backend database.

Please note that this is not the rule the CMS has to fulfill in order to work.
A lot of wordpress or drupal powered websites doesn't use generator meta tag
and they work like a charm.

We will spot for this information and if we're lucky enough we can have the
backend software auto detected simply looking at the HTML code.

As a bonus, we may want to fingerprint also the backend webserver family
looking at the HTTP response [Server](http://www.http-stats.com/header/Server)
header or [X-Powered-By](http://www.http-stats.com/header/X-Powered-By).

Those headers can be either not present so our autodetect would fail and we
eventually have to recover using [venerable scan and service detection techniques](http://nmap.org/book/vscan.html).
Remember, the goal here is gain as much informations as possibile with a single
GET HTTP request in a script.  We don't want to be intrusive during our
detection.

> Please note that this is not the rule the CMS has to fulfill in order to work.
> A lot of wordpress or drupal powered websites doesn't use generator meta tag
> and they work like a charm.

## Let's start with the ruby gem skeleton

To start with the [gengiscan](https://github.com/thesp0nge/gengiscan) ruby gem
skeleton I used the _bundler way_ to create a gem as explained in this
[blog](http://rakeroutes.com)
([here](http://rakeroutes.com/blog/lets-write-a-gem-part-one/) and
[here](http://rakeroutes.com/blog/lets-write-a-gem-part-two/)).

So the first stage was to setup [rvm](http://rvm.beginrescueend.com/) creating a gemset for my project so to
leave my ruby environment as clean as possible.

```
#  creating a gemset for gengiscan

$ rvm use 1.9.3
$ rvm gemset create gengiscan
$ rvm use 1.9.3@gengiscan
```

Now inside my gemset I will install [bundler](http://gembundler.com/) rubygem, to me the best piece
of code since the age of venerable [make](http://www.mtsu.edu/~csdept/FacilitiesAndResources/make.htm).

```
# installing bundler rubygem
$ gem install bundler
```

Now with bundler installed, you can ask him to create your rubygem skeleton,
the gengiscan one in my case.

```
# create your rubygem skeleton with bundler
$ bundle gem gengiscan
```

[github](http://github.com) repository is not here, so we must create it and
then set your remote origin to point to your archive.

```
# setting up git repository remote
$ git remote add origin https://github.com/thesp0nge/gengiscan.git
```

Now it's time for the first commit.

```
# your first commit
$ git commit -a -m 'First commit of gengiscan'
```

If you need a good explaination about how setting up your project dependencies
you can check [this Yehuda Katz](http://yehudakatz.com/2010/12/16/clarifying-the-roles-of-the-gemspec-and-gemfile/)
blog post about it.

Make sure to follow instruction you can find
[here](http://rakeroutes.com/blog/lets-write-a-gem-part-two/) and setup rspec
for your gem.

Now you're ready to go and you can start by writing your tests first.

## TDD workout

[webmock](https://github.com/bblimke/webmock/) is a rubygem that can be used
writing test mocking up target web server behaviour. In a simple way you can
stub a request for a specific URL specifiyng the requests headers and telling
what the response should have both in body than in the headers.

My first idea was to use [mechanize](http://mechanize.rubyforge.org/) to handle
the HTML parsing and retrieve the informations I need but, mechanize and
webmock don't fit so well.

The problem was that mechanize agent get method didn't honor webmock disabling
outbound HTTP connection so causing the testing framework to raise an
exception.

Ruby standard Net::HTTP API behaves completely different honoring webmock
stubbed request and fulfilling the tests.

``` ruby gengiscan dependencies as you can find in the gemspec
  gem.add_dependency('nokogiri')
  gem.add_development_dependency('rake')
  gem.add_development_dependency('rspec')
  gem.add_development_dependency('webmock')
```

So the testing framework changed the underlying architecture. True to be told,
this is the first time testing is punching my decisions so hard in a software I
wrote.

gengiscan will be based on standard Net::HTTP ruby library and
[nokogiri](http://nokogiri.org/) used to parse the HTML. Mechanize uses
nokigiri as well, so it sounds a feasible solution to use it raw over the
response HTML code.

``` ruby gengiscan spec_helper.rb file
require 'gengiscan'
require 'webmock/rspec'
```

A basic rspec test for gengiscan detecting the CMS declared in the mocked up response page.

``` ruby a very simple gengiscan spec
describe 'gengiscan' do
  let(:g) {Gengiscan::Engine.new}

  it 'is great' do
    stub_request(:get, "http://www.test.com/index.html").
         with(:headers => {'Accept'=>'*/*', 'User-Agent'=>'Ruby'}).
         to_return(:status=>200, :body=>"<!DOCTYPE html><html><head><meta name=\"generator\" content=\"WordPress 2.3\" /></head><body>A test cms powered</body></html>", :headers=>{})

    hash = g.detect('http://www.test.com/index.html')
    hash[:generator].should == "WordPress 2.3"

  end
end
```

The idea is that gengiscan API would give me back a ruby
[Hash](http://www.ruby-doc.org/core-1.9.3/Hash.html) with the detected
informations.

Of course as we write the spec first, running rake spec tests would fail giving
us the opportunity to code the detect API to let them pass

## The gengiscan API

I decided to move the URI to scan from the class initializer to detect method,
so in the latest version (0.30 as far as I'm writing this) you have to use the
API this way:

``` ruby using gengiscan
hash = Gengiscan::Engine.new.detect(ARGV[0])
```

hash would be a ruby Hash containing the HTTP status code, the server operating
system family, the X-Powered-by response value and the generator meta tag.

All the values I need comes from the HTTP response so I won't use nokogiri for
them but what Net::HTTP returned me. To get the Generator meta tag value I do
need to parse the HTML page.

``` ruby Gengiscan::Engine.detect method
def detect(url)
  uri = URI(url)
  res = Net::HTTP.get_response(uri)

  {:code=>res.code, :server=>res['Server'], :powered=>res['X-Powered-By'], :generator=>get_generator_signature(res)}
end
```

Nothing magic here, we ask for the web page using Net::HTTP from ruby standard
library and we take some values from the response we have.

Generator meta tag is retrieved using the get_generator_signature private method

``` ruby Gengiscan::Engine.get_generator_signature (private)
private
def get_generator_signature(res)

  generator = ""
  doc=Nokogiri::HTML(res.body)
  doc.xpath("//meta[@name='generator']/@content").each do |value|
    generator = value.value
  end

  generator
end
```

If we're lucky enough and the developer made his job the right way, there will
be only one meta generator tag in the HTML code we get it as response,
otherwise gengiscan will take the last appearing in the
[DOM](http://www.w3schools.com/htmldom/default.asp).

We need no more. And the following are some examples gengiscan detecting backend CMSes:

```
# gengiscan at work
$ gengiscan http://fireapp.handlino.com/
{:code=>"200", :server=>"nginx/1.0.15 + Phusion Passenger 3.0.12 (mod_rails/mod_rack)", :powered=>"Phusion Passenger (mod_rails/mod_rack) 3.0.12", :generator=>""}

$ gengiscan http://www.lapinella.com/
{:code=>"200", :server=>"Apache/2.2.14 (Ubuntu)", :powered=>"PHP/5.3.2-1ubuntu4.15", :generator=>"WordPress 3.3.2"}

$ gengiscan http://techcrunch.com/
{:code=>"200", :server=>"nginx", :powered=>nil, :generator=>"WordPress.com"}

$ gengiscan http://armoredcode.com
{:code=>"200", :server=>"nginx/1.0.14", :powered=>nil, :generator=>""}
```

As you may notice, we don't have always all the information we would have but
we automate the fingerprint step with a very simple ruby code that can be
reused as much as we like.

## Wrap up

Something to remember is that:

* fingerprint using a single HTTP GET **may be not** as accurate as a full
  intrusive scan with nmap (or the scanning tool you like most);
* servers may hide some informations so our fingerprint attempt will fail and
  we must recover surfing the app, looking at the web page extensions, looking
  in the HTML comments, ...
* ruby is fun to be used as programming language for security tool, it's HTTP
  API is damn good, it is built for the web, no stories;
* you don't want to go further in evaluating web site insecurities unless
  you're authorized to. Playing fair it is the first rule here on my blog.

Of course gengiscan can be improved with more robust error checking and more
fine granuled fingerprinting techniques.

But we will look at this in an upcoming project you will know more about it if
stay tuned over [armoredcode.com](http://armoredcode.com)
