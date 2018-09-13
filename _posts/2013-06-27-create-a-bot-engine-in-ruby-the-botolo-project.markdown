---
layout: post
title: "Create a bot engine in Ruby: the botolo project"
date: 2013-06-28 08:00
comments: true
published: true
featured: false
categories: ruby h4f hacking bot api gem rubygem codesake
thumb: walle.png
level:
hn: 
rd: 
---

There are a lot of tutorials for creating bots on several programming languages
and for a lot of web applications.

A bot is an autonomous piece of code, traveling the Internet executing one or
more tasks. A couple of days ago I was creating from scratch a bot for
[@codesake](https://twitter.com/codesake) to accomplish this task: _listen the
tweets for #scanwithdawn hashtag and capture the URL if given_

After I started I found that my engine can be extensible enough to be used
elsewhere.

Say hello to [botolo](https://github.com/thesp0nge/botolo).

<!-- more -->

## Being inspired

It has been started as a just for fun project meanwhile I was collecting
energies for working on [dawn](http://rubygems.org/gems/codesake-dawn) version
0\.80.

I googled a bit to find something already cooked but I didn't find anything
feasible to me. 
The only code I found was the [flamingo](https://code.google.com/p/flamingo/)
bot and I started looking at it for inspiration.

## Basic features I need

The bot I need for dawn must:

* be able to authenticate to twitter
* be able to search for an hashtag
* capturing the tweet text
* looking for an url
* start a code review with dawn
* go to sleep
* iterate forever

This means that I must write a code that:

* uses a configuration file mainly for twitter secrets storage
* a basic task DSL to describe the action the bot must carry on and the
  schedule
* implement the logic of the actions

While working on the first bot version I realized that all the specific logic
my [codesake-dawn](http://rubygems.org/gems/codesake-dawn) needs can be saved
in a single ruby file meanwhile all the engine logic can be general enough to
be reused in different bots.

That's why I started the [botolo](https://github.com/thesp0nge/botolo) project.

## What a botolo bot is?

Before start talking about technical details, we must define what is a bot
designed to be run using botolo.

{% blockquote %}
A bot designed for botolo engine is a ruby file with all bot tasks
implementation and a YAML configuration file. In the YAML configuration file
there is the tasks list and their schedule.
{% endblockquote %}

Looking to venerable C programming language, you may think the YAML file is the
.h include file defining function prototypes and the ruby file is the .c file
containing functions' implementation.

## Botolo: configuration of a dummy bot

Botolo configuration is a _per-bot_ YAML configuration file as said before. In
the configuration file there are three main parts:

* bot: general information about the bot and also the name of the ruby
  behaviour file
* twitter: twitter secrets to be used by the bot to interact with this social
  network. As the time I'm writing this, botolo is able to run only bots
  interacting with [twitter](https://twitter.com). 
* task: this the list of all the routines the behaviour file must implement.

Let's give a look to a basic YAML configuration file:

``` 
verbose: true

bot:
  name: mydummy-bot
  version: 1.0
  email: bot@codesake.com
  # This overrides any behaviour file passed as argument
  behaviour: behaviour.rb

twitter:
  name: codesake
  consumer_key: register an application to dev.twitter.com and put the value here
  consumer_secret: register an application to dev.twitter.com and put the value here
  oauth_token: register an application to dev.twitter.com and put the value here
  oauth_token_secret: register an application to dev.twitter.com and put the value here

task:
  - { schedule: every 10 m, action: hello_world }
``` 

This bot is named _mydummy-bot_, every ten minutes its task is descrived in the
hello\_world method you can find in the behaviour file that is mydummy-bot.rb.
Botolo engine expects to find the behaviour file in the same config file
directory, so there is no need to specify the ruby file at command line.

## Botolo: the dummy bot behaviour

There is a single but **critical** rule in writing the bot behaviour. The class
must be Botolo::Bot::Behaviour. I know, this breaks ruby naming convention if
you named the file with your bot name. 

I suggest you naming the config file with the name of your bot (and the YAML
extension of course) and naming the behaviour file as behaviour.rb so you will
keep safe ruby naming convention.

Another important rule is the class initialize method that it must accept an
Hash parameter.

``` ruby the initialize method skelethon
def initialize(options={})
  # put your code here
end
``` 

Following the aforementioned rules, our dummy\_bot behaviour file is:

``` ruby the behaviour.rb dummy-bot behaviour file
module Botolo
  module Bot
    class Behaviour

      def initialize(options={})
        @name = options[:name]
      end

      def say_hello
        puts "Hello world from #{@name}"
      end

    end
  end
end
``` 

It's pretty simple to create a bot for the botolo engine, now let's look at the
engine internals.

## Botolo: the engine initialization

Botolo engine initialization performs two major tasks:

* it reads the bot YAML file, building an hash with all configuration values
* it loads the behaviour file and creating an instance of
  Botolo::Bot::Behaviour

Embedded in the engine there is a dummy behaviour class performing no tasks but
placed here just for sake of playing well in case of errors or missing
filename.

``` ruby the engine loading the behaviour file
def initialize(options={})
  @start_time = Time.now
  @online = false
  @config = read_conf(options[:config])
  authenticate
  @tasks = @config['task']

  behaviour = File.join(".", @config['bot']['behaviour']) unless @config['bot']['behaviour'].nil? 

  $logger.helo "#{name} v#{version} is starting up"
  
  begin
    load behaviour
    $logger.log "using #{behaviour} as bot behaviour"
    @behaviour = Botolo::Bot::Behaviour.new({:name=>name})
  rescue LoadError => e
    $logger.err(e.message)
    require 'codesake/bot/behaviour'
    $logger.log "reverting to default dummy behaviour"
    @behaviour = Botolo::Bot::Behaviour.new({:name=>name})
  end
end
```

Reading configuration is nothing but a plain YAML parsing:

``` ruby the read_conf method
def read_conf(filename=nil)
  return {} if filename.nil? or ! File.exist?(filename) 
  return YAML.load_file(filename)
end
```

## Botolo: the main loop

Botolo engine main loop:
* iterates each task 
* check if the behaviour implements the action described in the YAML
  configuration and, if yes, running the action
* going to sleep for the amount of seconds specified in the bot configuration


``` ruby the botolo main loop
def run
  $logger.log "entering main loop"
  while true
    @tasks.each do |task|
      begin 
      @behaviour.send(task["action"].to_sym) if @behaviour.respond_to? task["action"].to_sym
      rescue => e
        $logger.err "#{task["action"]} failed (#{e.message})"
      end
      sleep calc_sleep_time(task["schedule"])
    end
  end
end
``` 

## The codesake bot

[codesake-bot](https://github.com/codesake/codesake-bot) is very easy to run with botolo. 

``` 
$ gem install botolo
$ cd codesake-bot # where both config.yaml and codesake-bot.rb files are
$ botolo config.yaml
```


## Off by one

There is a major drawback in this first botolo implementation. It's single
threaded. With no parallelism, there is no chance in having actions executed at
the same time. They must be executed each task per time, so the schedule time
is not absolute to the bot start time but it's relative to the previous task.

Next botolo versions will implement threads and parallelism between bot tasks.

That's all!
