---
layout: post
title: "Build an API for fun with Grape"
date: 2012-07-24 08:39
comments: true
published: true
featured: false
categories: builders ruby api tdd bdd armoredcode h4f gengiscan
hn: http://news.ycombinator.com/item?id=4284762
rd: http://redd.it/x2b41
thumb: ape.jpeg
---

I always dreamt about an
[API](http://en.wikipedia.org/wiki/Application_programming_interface) powered
website for [armoredcode](http://armoredcode.com) community. I do think that
every website should publish some sort of API to use services.

That's why [APIs for armoredcode](http://api.armoredcode.com) are online. 


<!-- more -->

![]({{site.url}}/images/grapes.jpeg)

I started last week hanging out on this project.
[Sinatra](http://www.sinatrarb.com/) is the _standard de facto_ choice for
building an API with ruby but, which framework to be used on top of this? Is
there out there something I can use to quickstart my project?

The answer is yes, it's called [grape](https://github.com/intridea/grape/).
Grape is a micro-framework built on top of Rack that provides a DSL to write
APIs.

So let's start the my first coding run. The goal is to implement an API to
provide [gengiscan](https://github.com/thesp0nge/gengiscan) as SaaS for people
don't want to install and run my gem on their machine.

## Layout

Source code layout is the standard Sinatra application. Some service files on
the root dir, the magic app.rb file, public, tmp and spec folders.

Gemfile is very basic. We will use gengiscan starting from version 0.40 that
introduces better error checking and we will use grape.
We will make TDD so we are going to use rspec and rack-test to write tests for
our API.

``` ruby Gemfile
source 'https://rubygems.org'

gem 'bundler'
gem 'grape'
gem 'json'
gem 'gengiscan', ">=0.40"
gem 'rake', :groups=>[:development, :test]
gem 'rack-test', :group=>:test
gem 'rspec', :group=>:test
``` 

Rakefile is in place just to provide rspec tasks. No great deals in this.

``` ruby Rakefile
#!/usr/bin/env rake
require "rspec/core/rake_task"

RSpec::Core::RakeTask.new

task :default => :spec
task :test => :spec
``` 

## TDD, let's do testing

Grape page at github is great in giving [help on using rspec](https://github.com/intridea/grape/#writing-tests). 

``` ruby spec_helper.rb
require "rack/test"
require './app'
``` 

As a behavior I want a [gengiscan api url](http://api.armoredcode.com/api/v1/gengiscan)
 that it will be called using POST HTTP method and the caller must give an host
parameter and an optional port in the HTTP POST request.

Of course, there is also an [hello world](http://api.armoredcode.com/api/v1/hello).

``` ruby api_spec.rb
require 'spec_helper'

describe ArmoredAPI do
  include Rack::Test::Methods

  def app
    ArmoredAPI
  end

  describe ArmoredAPI do
    describe 'GET /api/v1/hello' do
      it 'says hello to the world' do
        get "/api/v1/hello"
        last_response.status.should == 200
        JSON.parse(last_response.body)["hello"].should == "world"
      end
    end

    describe 'GET /api/v1/gengiscan' do
      it 'returns a 404 error if no host is provided' do
        get "/api/v1/gengiscan"
        last_response.status.should == 404
      end

      it 'run gengiscan over localhost' do
        post "/api/v1/gengiscan", "host"=>"localhost"
        last_response.status.should == 201
        hash = JSON.parse(last_response.body)
        hash["status"].should == "OK"
      end

      it 'run gengiscan over localhost on a port different than 80' do
        post "/api/v1/gengiscan", "host"=>"localhost", "port"=>"4000"

        last_response.status.should == 201
        hash = JSON.parse(last_response.body)
        hash["status"].should == "OK"
      end
    end
  end
end
``` 

I can improve my tests more mocking up a webserver, with plenty of response
using [webmock](https://github.com/bblimke/webmock/) I used for
[gengiscan](https://github.com/thesp0nge/gengiscan) but it will be a further
enhancement.

## The API source code

You can imagine a complex ruby application that makes all of these tests going green.
You're wrong. Just 26 lines of code including empty lines thanks to grape.

``` ruby app.rb
require 'grape'
require 'gengiscan'
require 'json'

class ArmoredAPI < Grape::API
  prefix 'api'
  version 'v1'

  get 'hello' do
    {:hello => 'world'}.to_json
  end

  desc 'Identify you host'
  namespace :gengiscan do

    post do
      host = "localhost"
      port = "80"

      host = request.params["host"] unless request.params["host"].nil?
      port = request.params["port"] unless request.params["port"].nil?
      Gengiscan::Engine.new.detect("http://#{host}:#{port}").to_json
    end
  end
end
``` 

With the prefix and version keywords I fixed that all the following url calls
start at '/api/v1'. 

Saying hello to the world is very easy as you may expect from a rack powered
application.

``` ruby the /hello api
get 'hello' do
  {:hello => 'world'}.to_json
end
``` 

``` ruby
$ curl http://api.armoredcode.com/api/v1/hello
{"hello":"world"}% 
``` 

Handling gengiscan is easy as well. You've just to take care about parameter in the request.
I used a namespace since I want a clear source code if I'll add (and I'll do) a
get handler explaining how to use the gengiscan API.

``` ruby Fingerprinting armoredcode.com
$ curl -d "host=armoredcode.com" http://api.armoredcode.com/api/v1/gengiscan
{"status":"OK","code":"200","server":"nginx/1.0.14","powered":null,"generator":""}%
``` 

``` ruby Fingerprinting armoredcode.com on a closed port
$ curl -d "host=armoredcode.com;port=3000" http://api.armoredcode.com/api/v1/gengiscan
{"status":"KO","code":null,"server":null,"powered":null,"generator":null}%       
``` 

That's all.
You can POST your requests to [gengiscan api](http://api.armoredcode.com/api/v1/gengiscan) specifying the host to
fingerpring in the host parameter and you've got an optional port parameter if
the server accept connection on non standard HTTP 80 port.

As far as today, making requests on HTTP is hardcoded so you can't fingerprint
an HTTPS powered server even mimic a connection on port 443.

``` ruby 
$ curl -d "host=gmail.com;port=443" http://api.armoredcode.com/api/v1/gengiscan
{"status":"KO","code":null,"server":null,"powered":null,"generator":null}% 
```

Enjoy it.
