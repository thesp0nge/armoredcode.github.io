---
layout: post
title: "How to generate bruteforce friendly strings"
date: 2013-11-18 07:59
comments: true
published: true
featured: false
categories: breakers owasp bruteforce regular-expressions string usernames passwords
thumb: strings.png
level:
hn: 
rd: 
---
It finally happened. You discovered that your favourite online store website
has a REST API to suggest usernames. It's a common pattern to allow the user
registration form to suggest alternatives username when the choosen one is
already taken. This feature is *so user-friendly*, *so userful* that almost
**every** project manager or sales representative will fight to have it online.

Most of times such kind of features allows attackers to enumerate legitimate
usernames making account password guessing easier.

<!-- more -->

## Security? I don't care about it

It's as usual first a software architecture issue rather than a technological
one. In order to give fancy (please read useless) services to users, you
exposes other customers to a potential attack.

The problem is... allowing people to choose a username, it gives the chance
this username is already taken. Wrong solution is to tell users (with an API
result code or a text string) the username already exist and eventually giving
free alternatives.

A clever approach is to use the user email address as username. Impressive
isn't it? Sometime sales and marketing people are not that clever... it's so
cute to allow users to choose their pet nickname as username... it makes... *so
community*.

## The bad

In this example, we will assume we are not that lucky. Username is fixed and it
was assigned us by the sysadmin. However a poorly security designed API exists.

* there's a REST API that, for a given username, it returns true or false if
  the username is contained in the customers database
* the API doesn't check it has been called by authenticated users
* usernames are 5 alphanumeric characters long

## The good

In order to ask the API for legitimate users we have two different approaches:

* generate _random_ usernames and iterate the process for a given number of
  times. This work if the codebase is big so it's luckly we will find usernames
  even with this non deterministic approach
* enumerate **all** possible usernames and iterate all the usernames. This is
  time consuming. There are 56 billions possible combination for a 5 letters
  long alphanumeric string. This is **huge**

First, we will build the regular expression representing a single character
space.

``` 
1.9.3-p429 :012 > [*('A'..'Z'),*('a'..'z'), *('0'..'9')]
 => ["A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R", "S", "T", "U", "V", "W", "X", "Y", "Z", "a", "b", "c", "d", "e", "f", "g", "h", "i", "j", "k", "l", "m", "n", "o", "p", "q", "r", "s", "t", "u", "v", "w", "x", "y", "z", "0", "1", "2", "3", "4", "5", "6", "7", "8", "9"]
```

Second, we will use the ```Array::sample()`` method to extract a sample array
build with this letter and number space. Since we need a string, we pack our
array in a string with the ```Array::pack()``` method.

It's pretty randomic to meet our purpose:

```
1.9.3-p429 :013 > [*('A'..'Z'),*('a'..'z'), *('0'..'9')].sample(5).join
 => "8tgxY"
1.9.3-p429 :014 > [*('A'..'Z'),*('a'..'z'), *('0'..'9')].sample(5).join
 => "b6Jrc"
1.9.3-p429 :015 > [*('A'..'Z'),*('a'..'z'), *('0'..'9')].sample(5).join
 => "17w3I"
1.9.3-p429 :016 > [*('A'..'Z'),*('a'..'z'), *('0'..'9')].sample(5).join
 => "s86pg"
1.9.3-p429 :017 > [*('A'..'Z'),*('a'..'z'), *('0'..'9')].sample(5).join
 => "iQoBw"
```

To enumerate all possible combinations you must calculate cartesian product
between those 5 arrays. This is a very CPU intensive task, bear in mind when
asking your cores to calculate this.

```
[*('A'..'Z'),*('a'..'z'), *('0'..'9')].product([*('A'..'Z'),*('a'..'z'), *('0'..'9')],
    [*('A'..'Z'),*('a'..'z'), *('0'..'9')],
    [*('A'..'Z'),*('a'..'z'), *('0'..'9')],
    [*('A'..'Z'),*('a'..'z'), *('0'..'9')])
```

The resulting array is 52 billions entries long. Consider this if you think
saving into a file. My suggestion is to iterate over it instead of saving.
