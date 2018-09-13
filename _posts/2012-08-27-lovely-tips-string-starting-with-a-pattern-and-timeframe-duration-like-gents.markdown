---
layout: post
title: "Lovely tips: string starting with a pattern and timeframe duration like gents"
date: 2012-08-27 09:24
comments: true
published: true
featured: false
categories: builders ruby hacking lovely-tips string regex regular-expression time time-difference duration
hn: 
rd: 
---

Life is too short to change framework or to learn a new programming language
only because there is no a shortcut to print out if a string starts with a
given pattern.

Just open the language class and make the method on your own like real men do.

<!-- more -->

## Is my string starting with the word 'foo'?

Of course this is a very interesting question and the following task solved
brilliant in many ways and so many times (I googled right before to submit this
post and I found tons of code snippets equals to the one I wrote. I won't win
Turing prize this year).

I'm writing a script connecting to our internal Vulnerability Assessment tool,
retrieving all the vulnerabilities and preparing some executive summary like
CSV files to help me in preparing my reports.

I have to gather together a list of IP addresses grouping by their pattern, but
unfortunately there is no method in the String class to tell if it's starting
with a given string.

Ruby is so kind to allow us to open the basic String object and add the methods
we need:

``` ruby The starts_with? method that won't make me win the Turing prize
class String
  def starts_with?(pattern)
    ! self.match(/^#{pattern}/).nil?
  end
end
``` 

## The import took (t1-t0) seconds... wait... what?

Second quick tips is about time. 

Sometimes you need to print how many seconds, or minutes or even hours a
particular task has been long to accomplish. In the script I was writing it was
the time the CSV importing method took to parse the file preparing the
aggregate data.

A difference between two Time objects it is a float, I opened the Numeric class
so to make the method available even for Integer numbers I used later in my
code.

``` ruby The duration method that allows me I write very inefficient code
class Numeric
  def duration
    secs  = self.to_int
    mins  = secs / 60
    hours = mins / 60
    days  = hours / 24

    return "#{days} days and #{hours % 24} hours" if days > 0
    return "#{hours} hours and #{mins % 60} minutes" if days == 0 and hours > 0
    return "#{mins} minutes and #{secs % 60} seconds" if days == 0 and hours == 0 and mins > 0
    return "#{secs} seconds" if days == 0 and hours == 0 and mins == 0
  end
end
``` 

Enjoy [lovely tips](http://armoredcode.com/blog/categories/lovely-tips)!

** UPDATE **

There was a typo in the duration method. I wrote minutes instead of mins. Fixed now

** UPDATE 2 **

Without the return keyword this method won't work at all. As you may understand I refactored this on the fly without testing. Shame on me.
