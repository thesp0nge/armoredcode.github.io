---
layout: post
title: "Parsing CVSS vector and publishing as API"
date: 2012-10-09 13:01
comments: true
published: true
featured: false
categories: builders api grape gengiscan cvss ruby rubygems opensource h4f
thumb:
hn: 
rd: 
---

Latest [July](http://armoredcode.com/blog/build-an-api-for-fun-with-grape/) I
wrote a post about having fun with [grape](https://github.com/intridea/grape/)
framework to build powerful APIs.

Today I used to put my [public api website](http://api.armoredcode.com) to the next level.

<!-- more -->

## Restyled

Today I finally configured an index.html page with some touch of twitter
bootstrap css and very basic information to quick start using the twos api I
published as far as today.

The idea is to put all useful [opensource code](https://github.com/thesp0nge) I
write on the Internet to be used as API.

## CVSS Parsing

Some days ago I faced a problem. A commercial IT solution with all it's bells
and whistles wasn't able to parse [CVSS](http://www.first.org/cvss) vector
string to tell me the vulnerability impact over my data.

So I decided to write a [small rubygem](https://github.com/thesp0nge/cvss) to
automate CVSS string parsing.

Parser is provided by a Cvss::Parser module and a parse method leaving all
other facilities as private.
For now only the base CVSS vector has been successfully parsed.

``` ruby Cvss::Parser
module Cvss
  module Parser

    attr_reader :base

    # It parses a string and it says if it's a good CVSS vector or not.
    def parse(string)
      @base = {}

      toks = string.split("/")
      return parse_base(toks)
    end


    private
    # AV:N/AC:L/Au:N/C:N/I:N/A:C
    def parse_base(tokens)
      return false if tokens.count != 6
      av = tokens[0].split(":")
      return false if av.count != 2 or av[0] != "AV" or (av[1] != "N" and av[1] != "L" and av[1] != "A")

      ac = tokens[1].split(":")
      return false if ac.count != 2 or ac[0] != "AC" or (ac[1] != "H" and ac[1] != "M" and ac[1] != "L")
      au = tokens[2].split(":")

      return false if au.count != 2 or au[0] != "Au" or (au[1] != "M" and au[1] != "S" and au[1] != "N")

      c = tokens[3].split(":")
      return false if c.count != 2 or c[0] != "C" or (c[1] != "P" and c[1] != "C" and c[1] != "N")

      i = tokens[4].split(":")
      return false if i.count != 2 or i[0] != "I" or (i[1] != "P" and i[1] != "C" and i[1] != "N")
      
      a = tokens[5].split(":")
      return false if a.count != 2 or a[0] != "A" or (a[1] != "P" and a[1] != "C" and a[1] != "N")


      
      
      @base = {:av=>av[1], :ac=>ac[1], :au=>au[1], :c=>c[1], :i=>i[1], :a=>a[1]}
      true
    end
  end
end
```

Some helpers were also provided to give some human friendly touch to the class:

``` ruby Cvss::Helpers
module Cvss
  module Helpers
    def data_integrity
      @base[:i]
    end
    def data_confidentiality
      @base[:c]
    end
    def data_availability
      @base[:a]
    end
  end
end
``` 

And the engine class is very basic:

``` ruby Cvss::Engine
require "cvss/version"
require 'cvss/parser'
require 'cvss/helpers'

module Cvss 
  class Engine
    include Cvss::Parser
    include Cvss::Helpers

  end
end
``` 

That's it so easy to use. You just use the parse method and then retrieve the
base vector stored as an Hash.

## Publishing it as an API

Publishing my CVSS parsing vector as API it's so easy with grape.

You just this code in your grape application file:

``` ruby My grape application
desc 'Parse CVSS vector'
namespace :cvss do
  post do
    vector = request.params["vector"] unless request.params["vector"].nil?
    cvss = Cvss::Engine.new
    cvss.base.to_json if cvss.parse(vector)
  end
end
``` 

And you can start parsing your CVSS vector now:
``` 
curl -d "vector=AV:L/AC:H/Au:N/C:C/I:C/A:C" http://api.armoredcode.com/api/v1/cvss
``` 

Enjoy it!
