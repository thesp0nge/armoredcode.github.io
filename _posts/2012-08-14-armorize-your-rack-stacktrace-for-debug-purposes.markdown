---
layout: post
title: "armorize your rack stacktrace for debug purposes"
date: 2012-08-14 16:08
comments: true
published: true
featured: false
tags: builders ruby ruby-on-rails rack sinatra padrino exception appsec owasp owasp-top-10 improper-error-handling
hn: http://news.ycombinator.com/item?id=4381300
rd: http://redd.it/y7emy
thumb: kaboom.png
---

Bugs? They happen. No one on Earth is smart enough to write a 100% bug free
piece of code. 
No matter how good are you, you're users still will try use your forms in an
unpredictable ways making your app to miserably fail.

Bugs, even serious security bugs can occur either in a piece of code wrote by
an #appsec specialist.

<!-- more -->

## The short one

Just to make internal application security workflow leaner, I wrote a [padrino powered](http://www.padrinorb.com) 
web application to help people in asking security assessments for servers and
web applications.

I also added a 500 internal server error handler to mail security team for the
bug stacktrace. Unfortunately something strange happened during vacations and
my User model screw up in a piece of code that it would be safe against null
pointer exceptions.

I don't know why but it seems that a variable has been found to be nil for a
race condition bug after a nil check already to be passed.

So the bug appeared after the logon page to be submitted and then people
passwords were disclosed to our internal mailing list.

** shame on me **

## Quickfix

Since bugs introduced from #appsec specialists are somewhat funniest to be
spotted from other people, I quickly fixed it in a couple of hours since the
first and only email it was disclosed.

But I wanted to draw something elegant... that's why I investigated further in
how to control the exception stacktracing in a rack application.

{% blockquote %}
Bugs, even serious security bugs can occur either in a piece of code wrote by
an #appsec specialist.
{% endblockquote %}

## The problem

The following was the original HTTP 500 error code handler that it was
suffering for sensitive information disclosure.

The webpage you see in development environment is rendered by the
Sinatra::ShowException class that it extends the Rack::ShowException class.

Both of them use a [Django inspired](http://djangoproject.com) layout that it
is hardcoded in the class.

``` ruby Original and bugged HTTP error code 500 hanlder
$showExceptions = Sinatra::ShowExceptions.new(self)

error 500 do
    @error = env['sinatra.error']
    html_body= $showExceptions.pretty(env, @error)
    # mail the error
end
``` 

In the template the following piece of HTML is executed:
``` html
<% req.POST.sort_by { |k, v| k.to_s }.each { |key, val| %>
<tr>
  <td><%=h key %></td>
  <td class="code"><div><%=h val.inspect %></div></td>
</tr>
<% } %>
```

Every parameter in the POST it is rendered in the resulting page, even the login password.
True to be told, Sinatra::ShowException class starts with the following preamble:

``` ruby 
# Sinatra::ShowExceptions catches all exceptions raised from the app it
# wraps. It shows a useful backtrace with the sourcefile and clickable
# context, the whole Rack environment and the request data.
#
# Be careful when you use this on public-facing sites as it could reveal
# information helpful to attackers.
``` 

But what do you have to do if you need the stacktrace to be printed out even in
a production environment without exposing sensitive informations?

Easy, just reopen the Sinatra::ShowException class and mask what you don't want
to show.


## The solution

First of all, I don't want to hardcode the keywords to hide in the source code,
that's why I want to add an Array to class constructor specifing which POST
parameters must be masked.

As a secondary behavior I want to exclude cookies from the HTML backtrace. This
must be a option choice to be specified while creating the runtime object.
Therefore using a ruby Hash to specify object options can be a quite flexible
solution.

``` ruby New Sinatra::ShowException constructor

module Sinatra
  class ShowExceptions
    attr_reader :options

    def initialize(app, options={:keywords=>[], :mask_cookies=>true})
      @options=options
      super(app)
    end
    
    # ...
``` 

``` ruby creating the object
$showExceptions = Sinatra::ShowExceptions.new(self, {:keywords=>['password'], :mask_cookies=>true)
``` 

The bizniz it is in the pretty method that it is called to render the exception
message applying the ERB template.

``` ruby New Sinatra::ShowException.pretty method
def pretty(env, exception)
  req = Rack::Request.new(env)
  post= req.POST
  if req.POST and not req.POST.empty?
    @options[:keywords].each do |k|
      if ! req[k].nil?
        req[k] = "**** masked ****"
      end
    end
  end

  if @options[:mask_cookies]
    req.cookies.each do |key, val|
      val = "**** masked ****"
    end
  end
  super(env, exception)
end
``` 

Using super() this way it means I recall original Rack::ShowException methods
after masking operations. Make sure that it fits your need before copycat this
class.

Enjoy!
