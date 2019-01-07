---
layout: post
title: "Using design by contract and TDD to enforce security: the coat project"
date: 2012-05-16 18:58
comments: true
published: true
featured: false
tags: coat ruby git github compiler lexer parser racc the-dragon-book builders tdd bdd design-by-contract
hn: http://news.ycombinator.com/item?id=3981041
rd: http://redd.it/tpowd
thumb: coat.jpg
---

## A small recap

In my [latest post](http://armoredcode.com/blog/is-by-design-by-contract-the-solution-for-safe-coding)
I introduced my summer project, the [coat](https://github.com/thesp0nge)
programming language.

The idea was to merge concepts from [design by contract](http://en.wikipedia.org/wiki/Design_by_contract) and
[BDD](http://en.wikipedia.org/wiki/Behavior_Driven_Development) or
[TDD](http://en.wikipedia.org/wiki/Test-driven_development) to build a
descriptive language to tell the world the constraint your code will fullfill.

Coat will be used to write ruby class skeleton with method pre and post
conditions, builtin data type checks, documentation and test cases.

Building a compiler is fascinating and a very self learning task but it's not that easy.

<!-- more -->

## The foundation

A couple of years ago I purchased [Marc-André](http://macournoyer.com/) 
[online book](http://createyourproglang.com/) about creating your programming
language from scratch.

I was working hard on the [Owasp Orizon](http://www.owasp.org/index.php/The_Owasp_Orizon_Framework) project and
I was frustrated about the parser I was building. It doesn't work at all (I
used the present tense since I never solved that issue, in fact that project is
almost orphaned).

I purchased a copy of [the dragon book](http://en.wikipedia.org/wiki/Compilers:_Principles,_Techniques,_and_Tools#Second_edition)
but my work didn't start.

So I purchased [Mark's book](http://createyourproglang.com/) that it seemed to
be more focused on practical examples on building a compiler.

I never applied all that stuff to Orizon, but I did some days ago when I started [coat](http://github.com/thesp0nge/coat).

## First task: create an hello world program

Created the repository the first task was to create an "Hello world" program. 

Coat is not a programming language designed to be translated in assembly and
then in an executable form, so I don't care about print something on standard
output. "hello\_world.coat" will describe a ruby class that puts "Hello world"
and it will create a test case ensuring that class methods will do an "Hello
world" message.

> Maybe this is not the most representative example of a class that needs
> security enforcements, however we need to start from something simpler to
> introduce coat grammar elements.

No words, just the code. Here is your "hello\_world.coat".

``` ruby
contract HelloWorld:
  pre:
    none
  post:
    write "Hello world" to stdout
  api:
    def say_hello:
      pre:
        none
      post:
        "Hello world"
``` 

## coat basics

Some language highlights before going deeper into details:

1. like python, coat relies on indentation level, there is no _end_ keyword
   that says a block of instructions is over
2. instructions are divided by newline
3. the ':' character makes a block of instruction to begin

A coat program is a contract between a class (a yet to be written HelloWorld
ruby class in this case) and the world. This contract says:

1. what the HelloWorld class does
2. the public api the HelloWorld class will expose

> I'm not interested in creating a language to tell programmers how to
> implement an algorithm. coat is about description and behavior, that's why
> HelloWorld class can have hundreds of private methods or it can implement
> say\_hello method in multiple ways. 
>
> The important bit is that it must print "Hello world" to standard output
> using the say_hello public api.

coat program starts with the contract name, that is a required information. 
This name must follow the ruby convention about camelcase formatting between
class and file names. 
So, the filename **must** be "hello\_world.coat" and the files the compiler
will create will be named: "hello\_world.rb" and "hello\_world\_spec.rb"

A contract can have class based pre and post conditions. 
Think at this scenario. You're building a proxy using WEBRick framework and you
want to honor the http\_proxy environment variable. 
If you translate this in coat, reading that variable from environment will be
your class pre-condition.

This way the HelloWorld class post condition is that in anycase, no matter if
in private methods the class will compute _pi_ up to 10 digits, or it make some
password cracking using rainbow tables, as output you will have "Hello world"
wrote to standard output.

Please note that _write_, _to stdout_ are both keywords saying that there'is an
expected output on screen. When files will be supported too, you will find
something like "write _something_ to _filename_"

### Your class public API exposed

After class pre and post conditions there is the block where all the magic
happens: the public API declaration.

Here you must declare all your public methods with their own pre and post conditions.
Please bear in mind that you're the boss, so if you want to add other public
methods to your class invalidating the contract you can. coat won't introduce
runtime checks, we're in a free world so you may decide later to write a
different code than the one the compiler mocked up or even leave BDD and TDD
for your code.

The method declaration is very _rubish_ and it follows the same scheme applied
to contract. One or more pre-conditions saying which constraints your method
parameters have and one or more post-conditions saying the expected values or
actions.

``` ruby
def say_hello:
  pre:
    none
  post:
    "Hello world"
``` 

coat would produce a method skeleton like:

``` ruby
def say_hello
  pre(nil)
  ret= "" 
  # put your code here

  post(ret, "Hello world")
  ret
end
``` 

pre and post will be class extensions I will add extending Class and they would
make some rspec like checks.

A more complex example can be something like:

``` 
def read_value_from_form:
  pre:
    param name must be a String
    param zip must be an Integer and a zip_code
  post:
    write User to table Accounts
    write "success" to stdout
``` 

In this scenario you're describing a public read\_value\_from\_form accepting 2
value as parameters: a string (name) and a zip code (zip). The idea is to apply
[Owasp Esapi for Ruby](https://www.owasp.org/index.php/Projects/Owasp_Esapi_Ruby) encoders to
params

coat would translate this contract definition this way.

```
def read_value_from_form(name, zip)
  pre(name, String.class)
  pre(zip, Integer.class)
  pre(zip, Owasp::Esapi::ZipCode.validate)

  ret = ""
  ret_save = false

  # put your code here

  ret_save = User.save
  post(ret, "success")
  post(ret_save, true)
end 
```

## State of art: what coat is able to do, today?

I started the project a week ago and there is **no** ruby code generation at
the moment. 
So no ruby class nor test case are generated so far.

At the moment, "hello\_world.coat" passes both the lexer than parser tests and
this is fine for the first post about coat.

Next actions must be of course ruby skeleton generation and implementing pre
and post Class extensions.

coat grammar is very unstable so you may expect some changes will happen almost
every day.

If you want to join the project, you can [fork the archive](http://github.com/thesp0nge/coat) 
or if you want to read about it don't miss the [posts I'm writing](/http://armoredcode.com/blog/categories/coat/) 
about coat.

Enjoy it!
