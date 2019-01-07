---
layout: post
title: "Create random keys in Ruby using SecureRandom"
date: 2013-10-15 08:10
comments: true
published: true
featured: false
tags: crypto appsec builders h4f securerandom ruby key owasp esapi oneliner
thumb: im-so-random.png
level:
hn: 
rd: 
---

Yesterday a friend of mine asked about truly random number generation in Java
and which are my thoughts about Random and SecureRandom classes.
Of course I told him to use
[ESAPI](https://www.owasp.org/index.php/Category:OWASP_Enterprise_Security_API)
calls since they are supposed to be robusts and well designed.

Apart from some weirdness in ESAPI.properties generation, ESAPI uses Java
native SecureRandom facilities, that turns the number generation to be truly
random in the computer science meaning.

Let's see how can we use Ruby SecureRandom class to create truly random keys to
use in our software.

<!-- more -->

## Clean SecureRandom API

[SecureRandom](http://www.ruby-doc.org/stdlib-2.0.0/libdoc/securerandom/rdoc/SecureRandom.html)
is a singleton class we can use in our software after requiring it. To
accomplish its job, SecureRandom will use:

* [OpenSSL](http://www.openssl.org) random\_bytes routing if OpenSSL has been
  installed in the machine.  It's not a dependency so, the related gem used to
  interface with operating system openssl facilities won't be installed;
* In Linux or Unix hosts, the /dev/urandom special file is been used.
  /dev/urandom is a truly random byte container full of entropy directly
  managed by the operating system kernel.
* On Windows machines, it uses the CryptGenRandom native method in Windows API

## SecureRandom and key generation

It's very common for a software to need a random key to be used, let me say as
salt in String.crypt method or in more sophisticated crypto routines. You may
want to have a random key just to use it as temporary password for your users.

We will use SecureRandom to create it one.

``` ruby
require 'securerandom'

def create_key(len=10)
  SecureRandom.hex(len)
end
```

Uh... is it possible just a silly method like that? Yes and not. You may
argued, at this point, that our method has a strong limitation: it gives you
and hexadecimal charset string, that means you can have [0-9a-f] characters
combination.

This is **not** a true 20 character random string, but looking at some
execution it can be safe enough for our purpose.

``` 
2.0.0p247 :009 > create_key
 => "bed9741125f193c0308f" 
2.0.0p247 :010 > create_key(20)
 => "19becd8719b6d8e9e2ff9c6c90921a08ce239376" 
2.0.0p247 :011 > create_key(20)
 => "9b177e74fbcf2ae59e7787677b92b93cc42459b3" 
2.0.0p247 :012 > create_key(20)
 => "9a618a7f2686e5e83376ed2c688151cd3d8f24dd" 
2.0.0p247 :013 > create_key(20)
 => "0a299b088442de6eb90956807011a765e24ce4ff" 
2.0.0p247 :014 > create_key(20)
 => "037b967086dbba0a014461106e03e1ed8ff5381c" 
2.0.0p247 :015 > create_key(20)
 => "3adb49097ae0c6d366e53965e45799e41dfc8bae" 
2.0.0p247 :016 > create_key(20)
 => "ab41776af357c42918660f484c11e0c2c2a2570b" 
``` 

## Off by one

A little cadeaux here.

If you're on a unix/linux machine, you can go and define a shell function to
read data from /dev/urandom for you.

``` 
function mkpw() { head /dev/urandom | uuencode -m - | sed -n 2p | cut -c1-${1:-10}; }

$ mkpw 20 
PuH/e4LIidEeiWQlGwBN
$ mkpw 20
QXpibAvzNAGm/6QfHRV9
$ mkpw 20
FXdZ/BcmzqQQ1gy624hu
$ mkpw 20
tJFqXLxy0NcSN6zJybUS
$ mkpw 20
qL3KR2L1huf0mkDQUTI/
```

Looking at the execution, this method is far better than the one described before.

Enjoy it!

_Image by [xkcd](http://xkcd.com)_
