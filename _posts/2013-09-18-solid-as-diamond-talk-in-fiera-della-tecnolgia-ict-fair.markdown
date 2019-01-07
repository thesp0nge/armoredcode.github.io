---
layout: post
title: "Solid as diamond talk in Fiera della tecnolgia ICT fair"
date: 2013-09-18 18:00
comments: true
published: true
featured: false
tags: ruby appsec builders h4f pentest-with-ruby broken-authentication-session-management xss information-gatering codesake
thumb: talk.png
level:
hn: 
rd: 
---

Today I delivered the "Solid as Diamond: use ruby in a web application
penetration test" talk in the [Fiera della tecnologia ICT](http://www.fdtict.it) 
fair in Milan, Italy.

It was supposed to be an not-too-much formal, innovative (as conference
template) fair in a too much business oriented scenario in ICT fairs in Milan.

The event missed some of its targets, but I will discuss in an _ad hoc_ post in
my [Italian blog](http://thesp0nge.com). 

The talk I delivered was the one I gave ad [Railsberry 2013](http://www.railsberry.com) in Cracow, April 2013.

<!-- more -->

## Talk outline

For both the events I used a broken web application written in
[Sinatra](http://www.sinatrarb.com). The code is of course opensource and
published on [this github repository](https://github.com/thesp0nge/railsberry2013).


``` ruby the vulnerable web application code
require 'sinatra' 

configure do 
  enable :sessions
end

get '/' do
  redirect '/login' 
end
get '/logout' do
  session[:logged] = nil
  redirect '/login'
end

get '/users' do
  erb :users
end

get '/users/:name' do
  redirect '/home' unless params[:name] == 'tom' or params[:name] == 'mark' or params[:name]=='john'
  redirect "/hello?name=#{params[:name]}"

end

get '/hello' do
  if ! session[:logged]
    @message="You must login first"
    erb :login
  end 

  @name = params[:name]
  erb :hello
end

get '/login' do
  @message=""
  erb :login
end

post '/login' do

  ret = check_creds(params["login"], params["password"])


  if ret == 0
    session[:logged]=params["login"]
    redirect '/home'
  end

  @message = "Wrong password for #{params["login"]} user" if ret == 1
  @message = "Unknown username #{params["login"]}" if ret == -1
  erb :login
end

get '/robots.txt' do
  headers "Content-Type"=>"text/plain"
  "User agent: *\nDisallow: /backend\nDisallow: /log\nDisallow: /db\nAllow: *\n"
end

get '/home' do
  if session[:logged].nil?
    @message="You must login first"
    redirect '/login'
  end 
  erb :home
end

get '/backend' do

  erb :backend
end

get '/log' do
  if session[:logged].nil?
    @message="You must login first"
    redirect '/login'
  end 

  erb :log
end

get '/db' do
  if session[:logged].nil?
    @message="You must login first"
    redirect '/login'
  end 

  erb :db
end

def check_creds(username, password)
  ret = 0
  users = File.readlines('db/creds.txt')
  users.each do |u|
    u=u.chomp
    return 0 if u.split(':')[0] == username and u.split(':')[1] == password
    return 1 if u.split(':')[0] == username and u.split(':')[1] != password
  end

  return -1
end
```

To the audience I saw that the only information they have is the url to attack ``` http://localhost:4567 ``` and eventually a valid username.

### Prerequisite

This assumption is valid for a pentest you deliver to your customer; to be more
effective you'll be given of a regular user.
In case you don't have it, you can eventually register a new one on the
website. Finally if no registration is possible you have to blind bruteforce a
valid username. Good luck.

### First goal is to leverage the attack surface and to understand the underlying technology

We saw that it's possible to fingerprint a web application technology seeing
how the answer you get back requesting a web page, let's say the website root.
To gather all the information you can, I wrote the [gengiscan gem](https://github.com/thesp0nge/gengiscan).

Running gengiscan specifying the target here it is the result:

```
$ gengiscan http://localhost:4567       
{:status=>:OK, :code=>"302", :server=>"WEBrick/1.3.1 (Ruby/1.9.3/2013-05-15)", :powered=>nil, :generator=>""}
``` 

So our target is a ruby written web application powered by WEBrick server.

We will get further information using the robots.txt file if available. I wrote
the [codesake_links gem](https://github.com/codesake/codesake_links) you can
use either to fetch a robots.txt looking for open urls and to bruteforce the
target with a given dictionary.

``` 
$ links -r http://localhost:4567                                   
[*] links v0.71 (C) 2013 - paolo@armoredcode.com is starting up at 16:59:42
16:59:42 [*] found 3 disallowed url(s) on http://localhost:4567
16:59:42 [*] /backend - 200
16:59:42 [*] /log - 302
16:59:42 [*] /db - 302
[*] leaving at 16:59:42
``` 

Using the robots.txt, it seems developers forget to place proper redirection on
the ``` /backend ``` url. This can be interesting in a second scanning step to
try to guess backend users weak passwords.

We will also use a url dictionary to find some other urls to use.

``` 
$ links -b support/test_case_dir_wordlist.txt http://localhost:4567
[*] links v0.71 (C) 2013 - paolo@armoredcode.com is starting up at 17:02:36
17:02:36 [*] http://localhost:4567/users: Open (7 msec)
``` 

As first step, we had that 

* technology used is ruby and we have also the application server vendor
  version
* we found /backend and /users links with just a couple of HTTP requests.

### Second goal is to find some regular users using user enumeration attack

Our web application [is vulnerable to user enumeration](http://armoredcode.com/blog/tales-from-a-login-page-intro/) since
it gaves two different error messages if the username is valid but the password
is not and in case both data are wrong.

I wrote a quick script available on the broken web application repository,
called ``` http_login_bruteforcer.rb ```. Check it out for futher details.
I used a regular username, tom as a canary and a dictionary of usernames you
can find everywhere in Internet.

``` 
$ ./http_login_bruteforcer.rb -P tom
[*] http_login_bruteforcer.rb v1.2.0 (C) 2013 - paolo@armoredcode.com is starting up at 17:07:24
17:07:24: PID - 97376
17:07:24: existing user tom used as canary
17:07:24: 500 words from dictionary loaded
17:07:24: awake... probing with: tom
17:07:24: awake... probing with: mark
17:07:24: awake... probing with: john
17:07:24: awake... probing with: terence
17:07:24: awake... probing with: root
17:07:24: awake... probing with: administrator
...
17:07:33: awake... probing with: liz
17:07:33: 5 user(s) found
17:07:33 [*] tom
17:07:33 [*] mark
17:07:33 [*] admin
17:07:33 [*] jason
17:07:33 [*] mary
[*] shutting down at 17:07:33

**Bingo**. 5 usernames found.
``` 

### Third goal is to exploit reflected cross site scripting in login and in hello pages

In hello page we noticed the name parameter you have in querystring is
reflected on resulting HTML. The same happens in the /login page with POST
parameters.

We will use [cross gem](https://github.com/thesp0nge/cross) to test for
reflected XSS.

``` 
$ cross -u http://localhost:4567/hello\?name\=paolo   
cross 0.35.0 (C) 2011, 2012 - paolo@armoredcode.com
Canary found in output page. Suspected XSS
``` 

```
$ cross http://localhost:4567/login                 
cross 0.35.0 (C) 2011, 2012 - paolo@armoredcode.com
Canary found in output page. Suspected XSS
```

In both cases appending a -D flag you will cause cross to dump the resulting
HTML in order to double check che canary presence.

## The slides

Slides are on [speakerdeck website](https://speakerdeck.com/thesp0nge/solid-as-diamond-use-ruby-on-a-web-application-penetration-test-for-www-dot-fdtict-dot-it)

<script async class="speakerdeck-embed" data-id="4587d71002a70131003a1257859247e3" data-ratio="1.2994923857868" src="//speakerdeck.com/assets/embed.js"></script>


_Image courtesy by [ikewinski](http://www.flickr.com/photos/ikewinski/)_
