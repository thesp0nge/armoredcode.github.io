---
layout: post
title: "H4F - invisible proxy... casper gem"
date: 2012-04-23 08:08
comments: true
published: true
tags: h4f ruby rubygem hacking attack breakers builders
hn: http://news.ycombinator.com/item?id=3877725
rd: http://www.reddit.com/r/programming/comments/snvhz/h4f_invisible_proxy_casper_gem/
---

Ruby is a great language for hackers and security researchers too. Of course
you can build amazing web applications using [Rails](http://rubyonrails.org) or
[Sinatra](http://sinatrarb.org) or even [Padrino](http://padrinorb.org)
frameworks. You can also build great tools using sophisticated APIs that make
very easy to craft HTTP requests, to intercept traffic, to run regular
expression or even to build a transparent proxy in very few lines of code.

<!-- more -->

## It all starts from a need...

As all stories go, I was in trouble with a web application during an
assessment. It was consuming bandwidth and I was pretty sure it made also some
ajax calls the developers refuse to admit.

Funny story, there are a lot of developers that are happy to hide some details
in order to save some recommendiation, actually you just make my work a little
harder and you make me upset.

So, there was me, my Safari browser and the target application. I was
investigating WEBrick for my own interested, so when it was clear I needed a
transparent proxy to gather all the traffic from Safari to webapp, I didn't
think about firing up another tool, but opened vim and cooked one in minutes.

## A word about reusing

Before going further you may think I'm fool to start a transparent proxy from
scratch instead of using one from Owasp Zap, or Paros. 

You're absolutely right. True to be told, I would have started suche kind of
project anyway somewhere in the future since I need something very custom for
my own usage.

I find that all proxies out there are memory consuming, most of them build in
Java technology with an old looking GUI. Another point is that gathered links
are hard to access to since there is no public API to call.

## What I need is...

Here we are. 

1. I need a transparent proxy (for the moment), collecting outgoing
   links in order to show me what my browser is doing in every moment with a
   timestamp associated.

2. Collected links must be accessible with an APIs and from the little binary
   tool.
3. The proxy code must be organized in APIs so to be integrated in other
   security tools

## casper rubygem

All the source code is on [this](https://github.com/thesp0nge/casper)
repository on github.

Building a proxy in ruby is somewhat trivial, all you have to do is to extend
the WEBrick::HTTPProxyServer

The config variable is the place when you put all the parameters you need to
pass to HTTPProxyServer when you will call the super class constructor.

Obviously, the most important parameter is the one telling the proxy _what_ it
has to to. 
In this case we tell the proxy to call a log\_requests routine to make all the
business.

``` ruby casper class initializer
def initialize(config={})
  @req_count = 0
  @hosts=[]
  config[:Port] ||= 8080
  config[:AccessLog] = []
  config[:ProxyContentHandler] = Proc.new do |req, res| 
    log_requests(req, res) 
  end

  super(config)
end
``` 

In the log\_requests there is no magic... we just dump the HTTP request with a
timestamp and we store the server into an hashmap so we can dump it when
requested.

``` ruby casper business logic module
private 
  def log_requests(req, res)
    $stdout.puts "[#{Time.now}] #{req.request_line.chomp}\n"
    if @hosts.index(req.host).nil?
      @hosts << req.host
    end
    inc_req_count
  end

  def inc_req_count
    @req_count += 1
  end
```

I put all the stuff in a private section to make sure you can't override the
default transparent behavior, in a first step.

## Signal handling

Using casper is easy as well.

``` ruby the casper binary
#!/usr/bin/env ruby
require "casper"

server = Casper::Proxy.new
trap("INT") { server.shutdown }
trap("INFO") { server.info }
trap("USR1") { server.dump }
server.start
``` 

Just create an instance of the Casper::Proxy class and start it. You have your
proxy running.

The tricky part here is that I wanted to play with signals with the _trap_ ruby
call.

As you may know, in a Unix environment there are a lot of signals you can send
to a process using the _kill_ system call. 

To learn more about the system call and the signal C library function:
```
$ man 2 kill
$ man 3 signal 
```

In the signal manpage there is also a list of signals you can send to a process
and even you can trap (most of them).

To be as compliant as possible to unix system programming default, I trapped
the SIGINT to the server shutdown call that is in the WEBrick::HTTPProxyServer
class and I trapped 2 custom signals to two different handlers.

SIGINFO will ask casper to print a one line statistic information row:

``` ruby casper info routine
def info
  $stdout.puts "[#{Time.now}] INFO #{@req_count} requests to #{@hosts.count} unique hosts"
end
```

SIGUSR1 will instead dump all visisted hosts observed by the proxy:
``` ruby casper dump routine
def dump
  $stdout.puts "Hosts we communicate with "
  if (@hosts.count == 0)
    $stdout.puts "None\n"

  else
    @hosts.each do |h|
      $stdout.puts " >>> #{h}\n"
    end
  end
end
```

## And now?

Casper::Proxy class is 50 lines of code counting newlines and comments. With 50
lines of code, thanks to the great WEBrick API you can build your own
transparent proxy.

However being transparent and gentle is not so funny is it?

Future enanchement will be:
1. saving session to SQL. Using an ORM like ActiveModel or Datamapper will make
   casper to be decoupled from DBMS so we may want to write some custom addon
   to use the user preferred DBMS.
2. we must implement a plugin system to make the user able to write some custom
   code and ask casper to execute it.
3. we must write some injection routine to try to help penetration tester in
   analysis
