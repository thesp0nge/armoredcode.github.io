---
layout: post
title: "Adding basic authentication support to wpscan"
date: 2012-10-23 09:10
comments: true
published: true
featured: false
tags: breakers ruby opensource hacking h4f wordpress wpscan basic-authentication http
thumb: wp-pumpkin.jpg
hn: http://news.ycombinator.com/item?id=4686955
rd: http://redd.it/11xoo1
---

[wpscan](http://www.wpscan.org) is an opensource tool designed to make
assessment over [wordpress](http://www.wordpress.org) installations.

Last week I had to evaluate the security level of an exposed wordpress
installation and I wanted to first try [wpscan](http://www.wpscan.org).
Unfortunately the target was published but password protected since it wasn't
publicly launched. [wpscan](http://www.wpscan.org) doesn't support basic
authentication out of the box but it's opensource and it's written in ruby... I
added it.

<!-- more -->

Working on opensource projects is really amazing. You can contribute adding
feature you need and if the maintainer doesn't think to include them in the
mainstream, then you can eventually use your own fork (using
[github](https://github.com) slang).

## A bit of theory

When a web resource is protected by basic authentication, the server answers with a 401 HTTP status code that it means **Non authorized** and with a **WWW-Authenticate** HTTP header specifying the authentication method it supports.

In the case of basic authentication you will see something like

```
WWW-Authenticate: Basic realm="My great protected resource"
```

The client pops out to the user a modal window asking for a username and a
password for the "My great protected resource" realm. Browser will create a
string with the username followed by a ':' character and then followed by the
password.

The aforementioned string is encoded using Base64 encoding function and sent to the server in an **Authorization** HTTP field.

```
Authorization: dGVzdDphcGFzc3dvcmQ= # => the username is test and the password is "apassword"
```

Then the server reverts the Base64 encoding making the credentials in a clear form and then it double checks with something stored in its filesystem.

> Base64 it is not an hashing function to protect web resources with a fair
> enough security level. Base64 must not used in any case as alternative to a
> password mechanism because it can be reverted in a clear text form with a
> single line of code.

As you can see, your secrets are not secured using Base64 encoding.

``` ruby
1.9.3p194 :003 > Base64.decode64("dGVzdDphcGFzc3dvcmQ=")
 => "test:apassword"
1.9.3p194 :004 >
```

## And a very bit of reality

However the wordpress powered site has its own password protected backend, the
basic authentication layer was there only to avoid google bot to start indexing
before real launch.

In this case, having a basic authentication in front of the webpage can be a
fair compromise for a couple of days.

The problem is that wpscan doesn't support basic authentication and since it's
a trivial ruby coding exercise I added it since I will face the same problem
again and again (making also a pull request to
mainstream repository).

## Patchworking the code.

First thing when you work on other people code is understanding how the code it
has been written. This is tough since you can't spend too much time in code
reviewing and understanding if you do need to launch your scan in minutes.

[wpscan](http://www.wpscan.org) source code is pretty organized. It can be
optimized in my taste and the whole tool can be packed in a rubygem and this
can be a further improvement in my repository fork.

I'll show git diff output command between the clean master and a branch I
created to add the basic authentication support. I like most working on a
separate branch following the workout described in
[this blog post](http://nvie.com/posts/a-successful-git-branching-model/)

### Let wpscan to understand the server needs credentials

First step, wpscan has a list of valid HTTP response code it can manage. We
have to say it that also 401 is a response code it can handle.

```
diff --git a/lib/wpscan/wp_target.rb b/lib/wpscan/wp_target.rb
index 59fa5a6..a3c335a 100644
--- a/lib/wpscan/wp_target.rb
+++ b/lib/wpscan/wp_target.rb
@@ -35,6 +35,7 @@ class WpTarget

   def initialize(target_url, options = {})
     @uri            = URI.parse(add_trailing_slash(add_http_protocol(target_url)))
+    @basic_auth     = options[:basic_auth]
     @verbose        = options[:verbose]
     @wp_content_dir = options[:wp_content_dir]
     @wp_plugins_dir = options[:wp_plugins_dir]
@@ -74,7 +75,7 @@ class WpTarget

   # Valid HTTP return codes
   def self.valid_response_codes
-    [200, 403, 301, 302, 500]
+    [200, 401, 403, 301, 302, 500]
   end

   # return WpTheme
```

Of course we will add a variable holding basic authentication parameter value
if it has been found. So it can be passed to inner methods.

### Create the command line flag

This is the wierdest part. wpscan handles command line using patterns I found
uncommons in ruby CLI utilities I hacked so far.

After adding the CLI option as the ones supported by GetOptLong, you have to
create appropriate setter method handling the parameter value.

Please note that we make also the encoding at this stage so that
options[:basic_auth] will hold (after it has been processed) the string we have
to put in the Authorization HTTP header as response.

```
diff --git a/lib/wpscan/wpscan_options.rb b/lib/wpscan/wpscan_options.rb
index 34942ca..5a28082 100644
--- a/lib/wpscan/wpscan_options.rb
+++ b/lib/wpscan/wpscan_options.rb
@@ -39,7 +39,8 @@ class WpscanOptions
       :wp_content_dir,
       :wp_plugins_dir,
       :help,
-      :config_file
+      :config_file,
+      :basic_auth
   ]

   attr_accessor *ACCESSOR_OPTIONS
@@ -48,6 +49,12 @@ class WpscanOptions

   end

+  def basic_auth=(basic_auth)
+    raise "Invalid basic authentication credentials" if basic_auth.index(':').nil?
+    @basic_auth="Basic #{Base64.encode64(basic_auth).chomp}="
+
+  end
+
   def url=(url)
     raise "Empty URL given" if !url

@@ -201,7 +208,8 @@ class WpscanOptions
         ["--follow-redirection", GetoptLong::NO_ARGUMENT],
         ["--wp-content-dir", GetoptLong::REQUIRED_ARGUMENT],
         ["--wp-plugins-dir", GetoptLong::REQUIRED_ARGUMENT],
-        ["--config-file", "-c", GetoptLong::REQUIRED_ARGUMENT]
+        ["--config-file", "-c", GetoptLong::REQUIRED_ARGUMENT],
+        ["--basic-auth", "-b", GetoptLong::REQUIRED_ARGUMENT]
     )
   end
```

### Make wpscan to raise error if 401 is received and no credentials passed

This is trivial. In the main program we have to raise an exception if the
target server answer us with a 401 status code and we didn't provide any
credentials to be used.

Next time you have to use newly provided '-b' flag baby.

```
diff --git a/wpscan.rb b/wpscan.rb
index 822daf2..f73a2fe 100755
--- a/wpscan.rb
+++ b/wpscan.rb
@@ -53,6 +53,10 @@ begin
     raise "The WordPress URL supplied '#{wp_target.uri}' seems to be down."
   end

+  if wp_target.has_basic_auth?
+    raise "The WordPress URL supplied '#{wp_target.uri}' seems to be protected with basic auth." unless wpscan_options.basic_auth
+  end
+
   redirection = wp_target.redirection
   if redirection
     if wpscan_options.follow_redirection
```

The helper method has_basic_auth? is defined in the web_site module in a very
simple code. If target answered 401 then it has been protected by basic auth
_(since I needed just basci authentication support, I generalized that a 401
status code required basic authentication without a deeper check in the HTTP
response headers for the required authentication mechanism)_.

### Authorize me

Last step: passing the Authorization HTTP header in response with the Base64
endoded credentials.

wpscan browser module uses Typhoeus::Hydra to make HTTP requests. Hopefully
this third party framework allows us to hack response parameters very easily.

```
diff --git a/lib/browser.rb b/lib/browser.rb
index 9daae9b..675ed3f 100644
--- a/lib/browser.rb
+++ b/lib/browser.rb
@@ -36,6 +36,7 @@ class Browser
   def initialize(options = {})
     @config_file = options[:config_file] || CONF_DIR + '/browser.conf.json'
     options.delete(:config_file)
+    @basic_auth = options[:basic_auth]

     load_config()

@@ -151,6 +152,17 @@ class Browser
       params = params.merge(:proxy => @proxy)
     end

+    if @basic_auth
+
+      if !params.has_key?(:headers)
+        params = params.merge(:headers => {'Authorization' => @basic_auth})
+      elsif !params[:headers].has_key?('Authorization')
+        params[:headers]['Authorization'] = @basic_auth
+    end
+
+
+    end
+
     unless params.has_key?(:disable_ssl_host_verification)
       params = params.merge(:disable_ssl_host_verification => true)
     end
```

## Wrap up

Now my own wpscan fork has a '-b' flag that it can be used in the command line
to specify username and password to enter basic authentication protected
wordpress installations.

```
$ ruby ./wpscan.rb -b test:apassword -u http://wp.test.blog
```

If you don't use the username:password form, the code will complain about the
credentials format and raising a runtime error stopping execution.

[wpscan](http://www.wpscan.org) is a great tool that it can be further improved
but it must be a part of any web application security specialist toolbox out
there.

_Image by [Eric M Martin](http://www.flickr.com/photos/ericmmartin/2986187518/in/photostream/)_

Enjoy it!
