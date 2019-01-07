---
layout: post
title: "Some security tips for ruby hackers: leveraging the attack surface: part 2"
date: 2012-06-27 07:57
comments: true
published: true
featured: false
tags: builders breakers ruby webapp gems pentest speech rubyday owasp owasp-testing-guide pentest-with-ruby
hn: http://news.ycombinator.com/item?id=4166118
rd: http://redd.it/vodso
---

In the [first part](http://armoredcode.com/blog/some-security-tips-for-ruby-hackers-leveraging-the-attack-surface-part-1/) of this overview about web application perimeter recognizance we stopped using [ciphersurfer](https://github.com/thesp0nge/ciphersurfer) to check for SSL certificate weakness.

Now we move further and we will prepare to make the real attack.

<!-- more -->

## A transparent proxy for all the needs

A transparent proxy can be very useful when you've to check your browser interaction with the website to attack.
Maybe we can discover some async call or how params are submitted and stuff like that. 

For sure you can use your browser inspector funtion (if you're using Apple Safari) or [firebug](http://getfirebug.com/) to interact as a web developers does.

Since I was learning more about networking Ruby APIs and in detail about
[WEBrick](http://apidock.com/ruby/WEBrick)  and how to use this API in a
security tool, I reinvented the wheel starting
[casper](https://github.com/thesp0nge/casper).

Creating a proxy with ruby and WEBrick is dramatically easy. You need just to
extend the HTTPProxyServer class and you're ready to go:

``` ruby 
require "webrick"
require "webrick/httpproxy"

module Casper
  class Proxy < WEBrick::HTTPProxyServer
    attr_reader :req_count
    attr_reader :hosts

    def initialize(config={})
      @req_count = 0
      @hosts=[]
      @urls=[]
      @trace_domain = ""
      @trace_domain = config[:trace] if config[:trace] and ! config[:trace].empty?

      config[:Port] = 8080 if ! config[:Port]
      config[:AccessLog] = []
      config[:ProxyContentHandler] = Proc.new do |req, res| 
        log_requests(req, res) 
      end

      super(config)
    end
``` 

In a private section I put business logic methods:

``` ruby
private 
def log_requests(req, res)
  if (@trace_domain == "") or ( ! req.request_line.index(@trace_domain).nil?)
    $stdout.puts "[#{Time.now}] #{req.request_line.chomp} => #{res.status}\n"
    $stdout.puts "---> #{req.body} #{req.request_method}" if req.request_method == "POST"
    if @urls.index(req.request_line.chomp).nil?
      @urls << req.request_line.chomp
    end
    if @hosts.index(req.host).nil?
      @hosts << req.host
    end
    inc_req_count
  end
end

def inc_req_count
  @req_count += 1
end
``` 

I find it useful to trace all POSTs my browser make during navigation in order
to inspect all values that can be tampered replying the submission...

As example the following trace showed my in a real world activity that browser
make a double submission over the login page submitting a lot of different
parameters, a cleartext password and some parameter like uuid suggesting some
code managing user ids is in place.

This can be interesting since I want to digg deeper during the tests trying to
make some privilege escalation attempt.

``` 
[2012-06-22 13:31:03 +0200] POST http://192.168.x.x/myapp/login HTTP/1.1 => 200
---> dtid=g7g71&cmd.0=onMove&uuid.0=z_7g_1&data.0=448px&data.0=246px&data.0=&cmd.1=onZIndex&uuid.1=z_7g_1&data.1=34&cmd.2=onClientInfo&uuid.2=&data.2=-120&data.2=1440&data.2=900&data.2=24&data.2=1196&data.2=688&data.2=0&data.2=0 POST
--------------------
[2012-06-22 13:31:17 +0200] POST http://192.168.x.x/myapp/login HTTP/1.1 => 200
---> dtid=g7g71&cmd.0=onMove&uuid.0=z_7g_1&data.0=448px&data.0=246px&data.0=&cmd.1=onChange&uuid.1=z_7g_6&data.1=admin&data.1=false&data.1=5&cmd.2=onChange&uuid.2=z_7g_9&data.2=admin&data.2=false&data.2=5&cmd.3=onOK&uuid.3=z_7g_1&data.3=13&data.3=false&data.3=false&data.3=false&data.3=z_7g_9 POST
``` 

## Check for backup files

You see it in TV movies. Detective looking into garbage to find informations
against bad guys.

In IT security some principles are still valid; sometimes you can find backup
files published on production servers full of useful informations like
passwords, ip addresses, source code, ...

Using [anemone](http://anemone.rubyforge.org/) it's easy to make some checks
for backup files presence.

``` ruby 
require 'anemone'
require 'httpclient'
h=HTTPClient.new()
Anemone.crawl(ARGV[0]) do |anemone|
  anemone.on_every_page do |page|
      response = h.get(page.url)
      puts "Original: #{page.url}: #{response.code}"
      response = h.get(page.url.to_s.split(";")[0].concat(".bak"))
      puts "BAK: #{page.url.to_s.split(";")[0].concat(".bak")}: #{response.code}"
      response = h.get(page.url.to_s.split(";")[0].concat(".old"))
      puts "OLD: #{page.url.to_s.split(";")[0].concat(".old")}: #{response.code}"
      response = h.get(page.url.to_s.split(";")[0].concat("~"))
      puts "~: #{page.url.to_s.split(";")[0].concat("~")}: #{response.code}"
  end 
end
```

In a real world attack we can go further behind this _gentle_ approach
computing the backup file url with brute force. However you must be careful
about making bursts of requests or you'll be busted by target web application
firewall or perimetral defence appliances.

## Wrap up

Make a deep and accurate recognition about a web application is a preliminary
step for a successful security test.

It's important that developers can make such tests not making assumptions about
the code that they have wrote.

On a production server, you must publish only files needed by the application
itself. No backups, no readme or other documentation, no too verbose config
files must be on the server.

This is not “Security through obscurity”, this is the least privilege principle.

## Off topic

A lot of ruby hackers attended my #rubyday talk last 15<sup>th</sup> June. I
prepared some very small demos for each piece of code or security tests I
presented in order to make them aware about that the topics I was talking about
do are real issues.

I performed well, after the talk I was very satisfied about the job done and
this doesn't happen too much often. I'm always very critical about myself as
speaker, but this time not. 

People liked it and it was the most upvoted talk for the conf.

The message they gave as a feedback is strong and clear:

* security do is a concern for us
* we don't know how to make good security
* please help developers

I can't wait more for other developer's conferences...
