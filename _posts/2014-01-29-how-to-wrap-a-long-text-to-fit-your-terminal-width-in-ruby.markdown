---
layout: post
title: "How to wrap a long text to fit your terminal width in ruby"
date: 2014-01-31 08:20
comments: true
published: true
featured: false
tags: builders how-to ruby text string wrap width
thumb: text.png
level:
hn: 
rd: 
---

Today I was working over a new tabular output for Codesake::Dawn and I faced a
problem. Vulnerabilities have a very long description that breaks all
formatting resulting in something unreadable.

The "Ruby Programming Language" doesn't help me that much. I wondered String
class already has something similiar but I was wrong. Also both _PrettyPrint_
and its releated _pp_ libraries didn't help me in breaking up a long text
justifying at a certain width.

Since it doesn't seem to be a difficult implementation, I spent a couple of
minutes reiventing (must check if I do reinvent it) the wheel.

<!-- more -->

## Justifying a text, the Ruby way

You have a very long text that breaks your terminal width resulting in a non elegant output. You have something like:

``` ruby
TEXT="Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat"
```

You want something more readable, like:

``` 
Lorem ipsum dolor sit amet consectetur adipisicing elit sed do eiusmod tempor
incididunt ut laboreet dolore magna aliqua Ut enim ad minim veniam quis nostrud
exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat
```

A quick and dirty hack that eventually it becomes the [justify
rubygem](https://rubygems.org/gems/justify), is to scan the text with a regular
expression looking for space separated words, and then to build a new string
with newline characters placed accordingly to the desired width.

``` ruby

class String
  def justify(len = 80)
    words = self.scan(/\w.+/)
    actual_len = 0
    output = ""
    words.each do |w|
      output += w
      actual_len += w.length
      if actual_len >= len
        output += "\n"
        actual_len = 0
      else
        output += " "
      end
    end
    return output

  end
end
```

The problem here is that every punctuation mark is deleted. The regex needs to
be really improved, however it fits my need of having the long vulnerability
description to be formatted with a certain width.

Since I modified the String class, when you include the _justify_ gem, all of
your strings inherits the justify method. Very easy.

```
puts TEXT.wrap(80)
puts TEXT.wrap(40)
```

Enjoy it!
