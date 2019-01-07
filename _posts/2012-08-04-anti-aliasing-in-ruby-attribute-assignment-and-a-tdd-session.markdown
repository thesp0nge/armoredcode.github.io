---
layout: post
title: "Anti aliasing in ruby attribute assignment and a TDD session"
date: 2012-08-04 14:50
comments: true
published: true
featured: false
tags: ruby cross builders tdd aliasing attribute appsec-in-august sea vacation
hn: 
rd: 
thumb: sea.jpg
---

Since a week far away I'm on vacation at Guardavalle Marina, in the very
Southern of Italy. Here I'm relaxing trying to fixing up some tool I use as
application security specialist at work.

I was working on [cross](https://github.com/thesp0nge/cross) and I faced a
strange aliasing bug while playing with an attribute assignment.
<!-- more -->

## The TDD workout

cross has very basic features. If you want to inject cross site scripting
attack patterns in the web application URL parameters, you can't choose the
parameter to inject. This is very limiting.

Using [Owasp WebGoat](http://code.google.com/p/webgoat/) as example, from a url obtained with an auxiliary parser,
starting from an url like the following:

``` 
http://localhost:8080/WebGoat/attack?Screen=130&menu=900
``` 

I would like to be able to say cross to inject attack vettors only on _Screen_
parameter leaving the other untouched.

This is the spec file I wrote.

``` ruby url_spec.rb
require 'spec_helper'

describe "Url fuzzer" do
  let (:url) {Cross::Url.new("http://localhost:8080/WebGoat/attack?Screen=130&menu=900")}
  it "will recognize the param list" do
    url.params.class.should == Array
    url.params.size.should == 2
  end

  it "will recognize parameter names" do
    url.params[0][:name].should == 'Screen'
    url.params[1][:name].should == 'menu'
  end

  it "will recognize parameter values" do
    url.params[0][:value].should == "130"
    url.params[1][:value].should == "900"
  end

  it "will provide a get shortcut for getting parameters value" do
    url.get("Screen").should == "130"
    url.get("menu").should == "900"
  end

  it "will handle errors smoothly" do
    url.get("nonexistent").should be_nil
  end

  it "will make a copy of parameters" do
    url.original_params.should == url.params
  end

  it "will make params Array to be reverted as string" do
    url.params_to_url.should == "Screen=130&menu=900"
  end

  describe "will provide an handy set shortcut that" do
    it "sets an existing params to a given value" do
      url.set("Screen", "123")
      url.get("Screen").should == "123"
    end

    it "handle the error condition smootly" do
      url.set("nonexistent", false)
      url.original_params.should == url.params
    end

    it "won't change the original params" do
      url.set("Screen", "123")
      url.original_params.should_not == url.params
    end
  end
  it "will fuzz" do
    url.fuzz("Screen", "12").should == "http://localhost:8080/WebGoat/attack?Screen=12&menu=900"
    url.fuzz("Screen", "afuzztest").should == "http://localhost:8080/WebGoat/attack?Screen=afuzztest&menu=900"
    url.fuzz("menu", "11").should == "http://localhost:8080/WebGoat/attack?Screen=afuzztest&menu=11"
  end

  it "will fuzz honoring original params if requested" do
    url.fuzz("Screen", "12").should == "http://localhost:8080/WebGoat/attack?Screen=12&menu=900"
    url.fuzz("Screen", "afuzztest").should == "http://localhost:8080/WebGoat/attack?Screen=afuzztest&menu=900"
    url.reset
    url.fuzz("menu", "11").should == "http://localhost:8080/WebGoat/attack?Screen=130&menu=11"

  end
end
```

Pretty easy, isn't it?!? This is the Cross::Url class I wrote to fit my needs.
As usual it's shorter the code than the tests even counting the whitespaces
(actually this code make all the tests to pass, let's go into the post seeing
the original class implementation).

``` ruby Cross::Url class
module Cross
  class Url
    
    attr_reader :url
    attr_reader :base_url
    attr_reader :params
    attr_reader :original_params

    def initialize(url)
      @url = url
      @params = []
      @original_params = []
      @base_url = url.split('?')[0]
      p_array = url.split('?')[1].split('&')
      p_array.each do |p|
        pp = p.split('=')
        param = {}
        param[:name] = pp[0]
        param[:value] = pp[1] unless pp[1].nil?

        @params << param
        @original_params  << param.dup
      end
      @original_params.freeze
    end

    def fuzz(name, value)
      set(name, value)
      "#{@base_url}?#{params_to_url}"
    end
    
    def get(name)
      value = nil
      @params.each do |p|
        value = p[:value] if p[:name] == name
      end
      value
    end

    def set(name, value)
      @params.each do |p|
        p[:value] = value if p[:name] == name
      end
    end

    def reset
      @params = @original_params
    end

    def params_to_url
      ret = ""
      @params.each do |p|
        ret += "#{p[:name]}=#{p[:value]}"
        if !(p == @params.last) 
          ret +="&"
        end
      end
      ret

    end
  end
end
```

## The antialias bug

Running the specs, results disappointed me.
![]({{site.url}}/images/alias_spec_fail.png)

It seemed to me I was encountering an alias bug for the @original_params class
attribute. 

What I was looking for was to save the original parameter values in an Array I
eventually can use again to replay the original request.

It seems that every change I was playing over the param Array it was reflected
also on original_params.

This is very weird since I made a dup for the params Array and then freezing
the original_params one.

I was arguing a day about this bug... well, in the time I was not swimming or
enjoying making castles in the sand with my child. I know you can understand.

``` ruby freezing the wrong object
def initialize(url)
  @url = url
  @params = []
  @original_params = []
  @base_url = url.split('?')[0]
  p_array = url.split('?')[1].split('&')
  p_array.each do |p|
    pp = p.split('=')
    param = {}
    param[:name] = pp[0]
    param[:value] = pp[1] unless pp[1].nil?

    @params << param
  end

  @original_params = @params.dup
  @original_params.freeze
end
``` 

{% blockquote %}
In Ruby, a variable is simply a reference to an object.
{% endblockquote %}

## Making the right clone

I tried so to make a copy of the variable I was putting in my "not to be
tampered" array and everything went well.

``` ruby freezing the right object
def initialize(url)
  @url = url
  @params = []
  @original_params = []
  @base_url = url.split('?')[0]
  p_array = url.split('?')[1].split('&')
  p_array.each do |p|
    pp = p.split('=')
    param = {}
    param[:name] = pp[0]
    param[:value] = pp[1] unless pp[1].nil?

    @params << param
    @original_params  << param.dup
  end
  @original_params.freeze
end
``` 

![]({{site.url}}/images/alias_spec_green.png)

## Fuzzing and inspecting

The method I will use most in Cross::Url it will be the fuzz() method. I
created this to be able to mangle the request URI injecting a Cross Site
Scripting attacking parameter into a specified parameter enumerating all the
possible attack scenarios.

``` ruby Cross::Url.fuzz()
def fuzz(name, value)
  reset
  set(name, value)
  "#{@base_url}?#{params_to_url}"
end
```

The idea is to call the fuzz method in a loop like the following one and then
attacking all the target page parameters:

``` ruby
url = Cross::Url.new("http://localhost:8080/WebGoat/attack?Screen=130&menu=900")

Cross::Attack::XSS.each do |pattern|
  url.params.each do |par|
    page = @agent.get(url.fuzz(par[:name], pattern))
    scripts = page.search("//script")
    scripts.each do |sc|
      found = true if sc.children.text.include?("alert('cross canary');")
      @agent.log.debug(sc.children.text) if @options[:debug]
    end
  end
end
``` 

