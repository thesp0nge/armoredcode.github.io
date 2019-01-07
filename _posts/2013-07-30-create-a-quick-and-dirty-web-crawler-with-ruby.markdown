---
layout: post
title: "Create a quick and dirty web crawler with ruby"
date: 2013-07-30 07:58
comments: true
published: true
featured: false
tags: dast ruby appsec crawling information-gathering pentest-with-ruby rubygems codesake
thumb: crawler.png
level:
hn: 
rd: 
---

A couple of days ago, I was starting a new security activity over a website I
never saw before. If you remember a last year
[post](http://armoredcode.com/blog/some-security-tips-for-ruby-hackers-leveraging-the-attack-surface-part-1/),
the first task is to crawl the website looking for intersting pages.

In that post example, crawler was very lame and basic. I need something more
sophisticated that eventually it will become part of the codesake project.

<!-- more -->

## Tell me more about the target

First version of the crawler just used
[anemone](https://rubygems.org/gems/anemone) rubygem to print page urls.

``` ruby the first version of my custome made crawler
require 'anemone'

Anemone.crawl(target) do |anemone|
  anemone.on_every_page do |page|
      puts page.url
  end
end
``` 

Of course this works but I have no information about page content or eventually
if the website answer with a particular error code to my requests.

I needed to upgrade my code with three basic functionalities:

* basic HTML parsing to find forms in the retieved web page
* HTTP return code handling
* persistence: I need to save my data in a usable format

I also told anemone to stop at site depth equal to 2. Of course this can be
easily turned into a command line parameter.

It all starts with:

``` ruby the new crawler anemone loop start
Anemone.crawl("#{site}", :discard_page_bodies => true, :depth_limit=>2) do |anemone|
  anemone.on_every_page do |page|

  ... 
``` 

## Basic HTML parsing

After I retrieved the page I wrote two small routines to read HTML body either
in case of HTTP or HTTPS protocol. This is a quick and dirty code, please allow
me not to having spent too much time in refactoring.

```ruby basic html parsing for my crawler

res = read_http(page.url)   if page.url.instance_of?(URI::HTTP)
res = read_https(page.url)  if page.url.instance_of?(URI::HTTPS)

...

if res.code.to_i == 200
  doc = Nokogiri::HTML(res.body)
  puts "#{page.url} (depth: #{page.depth}, forms:#{doc.search("//form").count}) "
end
``` 

Of course I used [nokogiri](https://rubygems.org/gems/nokogiri) gem to parse
the HTML code. The pieces of code reading the page are not that kind of magic. 

I broke into 2 routines mostly for not to deal with too complexity in a single
piece of code. You can improve this code in tons of ways.

``` ruby the two routines reading the page
def read_http(url)
 uri = URI(url)
 Net::HTTP.get_response(uri)
end

def read_https(url)
  response = nil
  uri = URI(url)

  http = Net::HTTP.new(uri.host, uri.port)
  http.use_ssl = true
  http.start do |http|
    response = Net::HTTP.get_response(uri)
  end
  response
end
```

## HTTP response code handling

I need to now if a referenced page is a broken link or if it raises an
application error. This can be some old code no more referenced or buggy or
whatever.

Even more interesting is if the page redirect me to another location.
Eventually I will find another url this way.

I also prompted the user if the website is asking me credentials with basic
authentication. An interesting place to try to bruteforce some very well known
user's password.

``` ruby http response code handling in my crawler loop 
puts "#{page.url} is a redirect to #{res['location']}" if res.code.to_i == 301

if res.code.to_i == 200
  doc = Nokogiri::HTML(res.body)
  puts "#{page.url} (depth: #{page.depth}, forms:#{doc.search("//form").count}) "
end

puts "#{page.url} was not found"                if res.code.to_i == 404
puts "#{page.url} requires authorization"       if res.code.to_i == 401
puts "#{page.url} returns an application error" if res.code.to_i == 500
``` 

## Persistence

I do love [datamapper](http://datamapper.org) ORM so I used it to store
retrieved url in a SQLite database. 

``` ruby my datamapper model for urls
require 'dm-sqlite-adapter'

class Url
  include DataMapper::Resource

  property  :id,          Serial
  property  :url,         Text,       :required=>true
  property  :code,        Integer
  property  :redirect,    Text
  property  :depth,       Integer
  property  :forms,       String,     :length => 256
  property  :created_at,  DateTime,   :default=>DateTime.now
  property  :updated_at,  DateTime,   :default=>DateTime.now

end
```

``` ruby saving my urls if not already there
if ! Url.first(:url=>page.url)
  u = Url.new
  u.url = page.url
  u.depth = page.depth
  u.forms = doc.css("form").map{ |a| (a['name'].nil?)? "nonamed":a['name'] }.compact.to_s.gsub("\n", ",") unless doc.nil?
  u.code = res.code.to_i
  u.redirect = res['location'] if res.code.to_i == 301

  ret = u.save
  saved += 1 if ret
  if ! ret

    puts "#{page.url} not saved"
    u.errors.each do |e|
      puts " * #{e}"
    end
  end

end
```

## Off by one

Of course you can improve the crawler in millions of way. I will add some
information about the detected form and I will alert if an url will accept
parameters also in the query string.

Something interesting can be also detecting comments into HTML code and to grab
all javascript files referenced by the projects.

The new crawler I'm hacking on is hosted on
[github.com](https://gist.github.com/6054995) as usual. 

_Image courtesy by [Martin LaBar](http://www.flickr.com/photos/martinlabar/)_
