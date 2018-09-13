---
layout: post
title: "They are tracking at you - pt.1"
date: 2012-09-07 10:05
comments: true
published: true
featured: false
categories: breakers cookie http privacy zombie-cookie tracking spy
hn: 
rd: http://redd.it/zhxtl
thumb: zombie-cookies.jpeg
---
Cookies are often used from companies to store informations client side to
track people on their web sites.

In the name of the user privacy and preventing web site tracking, people
sometimes disable cookie saving or they install browser plugins to allow them
to anonymously surf the Web.
 
There is a technique to track the users even if they don't want: using a
[zombie cookie](http://en.wikipedia.org/wiki/Zombie_cookie).

<!-- more -->

For short, a zombie cookie is a particular cookie saved in all the available
browser storing mechanisms making users diffcult to delete them. 

Accordingly to [wikipedia page](http://en.wikipedia.org/wiki/Zombie_cookie), a website using zombie cookies will try to save data in the following storage:

* in browser standard cookies repository
* in browser web history
* the [ETag value](http://en.wikipedia.org/wiki/HTTP_ETag)
* in Internet Explorer [userData storage](http://msdn.microsoft.com/en-us/library/ms533007.aspx). However this won't work anymore starting from IE9
* HTML5 storages like: Session, Local, Global and Database using SQLite
* RGB values using PNGs generation via HTML5 Canvas
* [Local Shared Objects](http://en.wikipedia.org/wiki/Local_Shared_Object) using Flash
* Silverligth Isolated Storage
* A sort of syncing mechanism respawning cookies using JS.

[Evercookie](http://samy.pl/evercookie/) is one of the most famous JS framework implementing zombie cookie technique.

The [js source code](https://github.com/samyk/evercookie/blob/master/evercookie.js) it's
pretty complex, at least for me that I'm not a JS ninja, however it appears
well written and easy to study.

How to protect from a website tracking at you this way? It's hard if you have
been "infected" by a zombie cookie, you've got some work to do to clean your
environment from all this rubbish data.

## Detecting Evercookie framework

If you suspect a site from using very intrusive tracking techniques like zombie
cookie you can check with few lines of code.

Just ask Nokogiri to parse the HTML you retrieve from the suspected malicious
site and then check if some included script is named evercookie-or-something.

``` ruby a lame evercookie detector
require 'open-uri'      
require 'nokogiri'      

html = open(ARGV[0])
doc = Nokogiri.HTML(html) 
doc.css('script').each do |sc|
  puts "evercookie js framework detected" if sc['src'].include?("evercookie")
end
``` 

Of course there are many way to override this check, renaming the js is the first one.

In this case maybe you want to download the js file, calculating some hash over
it and compare the result with a precalculated hash values coming from zombie
cookie js frameworks like evercookie.

In upcoming posts I'll try to make a more robust script to test a website for
cookies it wants to store in our browsers.

For a security point of view, it can be a challenging tests for everyday
security activities.


Enjoy!
