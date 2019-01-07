---
layout: post
title: "Even before your secure coding... patch your server"
date: 2012-03-19 09:10
comments: true
published: true
tags: builders patching rdp ms12-020 monday-report
hn: http://news.ycombinator.com/item?id=3722646
---

## Monday security report

Last week was quite busy in term of security issues. While people were still
facing a security issue affecting [Ruby on Rails](http://rubyonrails.org)
applications and that was used to exploit
[github](https://github.com/blog/1068-public-key-security-vulnerability-and-mitigation),
a new serious vulnerability is the candidate to make pentester happy while
assessing customers systems: the
[MS12-020](http://technet.microsoft.com/en-us/security/bulletin/ms12-020)
bullettin.


<!-- more -->


## Microsoft MS12-020 bulletin

Last week [Microsoft](http://www.microsoft.com) released a security bullettin
that it's likely to gain notoriety among pentester in next years: the
[MS12-020](http://technet.microsoft.com/en-us/security/bulletin/ms12-020)
bullettin.

Exploits [[1]](http://pastebin.com/jzQxvnpj),
[[2]](http://pastebin.com/4FnaYYMz), [[3]](http://pastebin.com/WYx9kRQ6)
appeared in the wild with an high burn rate. So you may think twice if you're
thinking about _not patching_ your server.

I know, you're a good guy and you don't expose **RDP** protocol on a
**Microsoft Windows** based server over the Internet, but there are also _I'm
in rush, I have to deploy_ guys who make mistakes and who don't take
architectural security so seriously.

## What about opensource?

Opensource software has security vulnerabilities as well, no one is perfect.
Someone will argument that opensource maintainers are quickies to fix
vulnerabilities compared to commercial vendors. 

Most of times this is true, however when a project don't have a strong
community behind, it fails to have a well organized security response team.

## NGINX memory corruption

This is not the case of [nginx](http://nginx.org) web server that soffered of a
memory corruption vulnerability. [Security advisories](http://nginx.org/en/security_advisories.html)
 told that latest stable version, _1.0.14 as the time we're writing_ is not vulnerable.

Looking at the [patch](http://nginx.org/download/patch.2012.memory.txt), it
seems that the usage of _ngx\_cpystrn_ exposes the code to a off by one buffer
overflow.

``` c nginx 1.0.14 memory corruption patch http://nginx.org/download/patch.2012.memory.txt
...
--- src/http/modules/ngx_http_fastcgi_module.c
+++ src/http/modules/ngx_http_fastcgi_module.c
@@ -1501,10 +1501,10 @@ ngx_http_fastcgi_process_header(ngx_http
                     h->lowcase_key = h->key.data + h->key.len + 1
                                      + h->value.len + 1;
 
-                    ngx_cpystrn(h->key.data, r->header_name_start,
-                                h->key.len + 1);
-                    ngx_cpystrn(h->value.data, r->header_start,
-                                h->value.len + 1);
+                    ngx_memcpy(h->key.data, r->header_name_start, h->key.len);
+                    h->key.data[h->key.len] = '\0';
+                    ngx_memcpy(h->value.data, r->header_start, h->value.len);
+                    h->value.data[h->value.len] = '\0';
                 }
...
```

Last memory corruption vulnerability was found by [Matthew Daley](http://www.securityfocus.com/archive/1/521960) and fixed yesterday,
15<sup>th</sup> March. Today I applied the same patch on the server serving
[armoredcode.com](http://armoredcode.com) as well.

Advisory says that:

> Without this fix contents of previously freed memory might be sent to
> a client if an upstream server returned specially crafted response,
> potentially resulting in sensitive information leak.

There is no magic bullet in application security. The important thing is that a
server is a living object and that vulnerabilities appear almost everyday. When
you expose a web application to your customers (it doesn't matter if using
Internet or an intranet) you must take care about system patching.

You don't want to spend countless hours in safe coding, testing and code review
and having all your system being exploited by an old school buffer overflow, do
you?
