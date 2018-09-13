---
layout: post
title: "H4F - use robots.txt as a weapon with links rubygem"
date: 2012-04-06 09:00
comments: true
published: true
featured: true
categories: h4f ruby rubygem hacking builders breakers source git robots.txt links
hn: http://news.ycombinator.com/item?id=3806318
rd: http://www.reddit.com/r/netsec/comments/rw099/h4f_use_robotstxt_as_a_weapon_with_links_rubygem/
---

Did you ever think about how much information did you disclose when you publish a website?
In order to control how the site will appear in search results, webmasters
create a [robots.txt](http://en.wikipedia.org/wiki/Robots_exclusion_standard)
file telling crawlers what they have to consider in their indexing quest and
which urls they must ignore so search engines won't show in search results.

But robots.txt is accessible also by humans that may decide to override your _Disallow_ clauses.

<!-- more -->

## Information gathering is fun again

This the robots.txt for [armoredcode](http://armoredcode.com/robots.txt)
website. It's easy, there are no secrets here so I tell any crawler is coming
here to please visit and index all the links.

{% highlight sh%}
User-agent: *
Allow: /
{% endhighlight %}

![]({{site.url}}/images/a_robots_txt.png)
For bigger sites with multiple sections things can be more interesting. 

Luckly we **don't want** to make troubles but we're curios and there is nothing
weong in looking in a public available text file. The important thing is that
you (webmaster) don't forget to protect directories for unauthorized access.

![]({{site.url}}/images/one-directory.png)

I know it seems to weird that people wrote a directory in a robots.txt but the
same directory is browsable by anyone, but you know... the Net is a strange
place and errors can easly occur so you may find a wordpress include directory
you can navigate.

Of course in this case, web server will recover the error parsing PHP code
before disclosing source code or connection parameters but in other situation
people is not so lucky.

![]({{site.url}}/images/open-dir-2.png)

You may find cache files that are full of interesting information or even... databases.

![]({{site.url}}images/db-over-the-net.png)

## Using robots.txt to fingerprint your backend

It's easy to see that a **lot** number of websites doesn't delete installation
files or common directories. They simply put all the information on the
directory serverd by web server and start publishing content relying only to
robots.txt to hide some information.

This behaviour can help an attacker instead to exactly fingerprint your backend
technology without even a portscan, just by looking at the robots.txt instead.

![]({{site.url}}images/dot-net-fingerprint.png)

Looking at the /\_vti\_ directories it's easy to spot a Microsoft .NET powered
website, therefore served by a Microsoft Windows operating sytem. If we (as
attackers) are lucky enough, those directories change in the time so it will be
also possible to fingerprint the exact .NET framework version looking at the
directory that are in place. 

[Drupal](http://drupal.org/) and [wordpress](http://wordpress.org) are both
easy to fingerprint by looking at the robots.txt file content.

![]({{site.url}}/images/fingerprint-drupal.png)

The former can be detected by a lot of text files that sometimes are still
available discosing the exact version number.
![]({{site.url}}/images/fingerprint-by-robots.png)
The latter is easly detected by the wp-_something_ directories that need to be
disallowed from indexing.

Fingerprint the exact technology you're using can give an attacker further
information to fine tune more attacks.

Please note that I'm note saying that [security through obscurity](http://en.wikipedia.org/wiki/Security_through_obscurity) is good.
I'm just saying that you have to remove all the files and all the directories
that you don't need instead of thinking that make them invisible from spiders
will be enought.

## Let's the code talk: the links rubygem

As a penetration tester my concern is to gather as much as information as
possible possibly without making too much noise.

That's why I wrote a script to automate the robots.txt file scanning. The
script eventually evolved in a full featured project and it evolved in a
published [rubygem](https://rubygems.org/gems/palco).

The idea is very simple. Taking an url, asking for the robots.txt and parsing
for disallow urls, then making the supposed disallowed urls http get and then
looking at the response code.

All the [links](https://github.com/thesp0nge/links) code is under the
Links::Api namespace.

First relevant method is the one that fetch robots.txt. I discovered a bug
while I was writing at this article but I disclosed it in awhile.

``` ruby Links::Api.robots - get the robots.txt file
def self.robots(site, only_disallow=true)

  if (! site.start_with? 'http://') and (! site.start_with? 'https://')
    site = 'http://'+site
  end

  list = []
  begin
    res=Net::HTTP.get_response(URI(site+'/robots.txt'))
    if (res.code != "200")
      return []
    end

    res.body.split("\n").each do |line|
      if only_disallow
        if (line.start_with?('Disallow'))
          list << line.split(":")[1].strip.chomp
        end
      else
        if (line.start_with?('Allow') or line.start_with?('Disallow'))
          list << line.split(":")[1].strip.chomp
        end
      end
    end
  rescue
    return []
  end
  
  list
end
``` 

The bug is that I check for _Allow_ and _Disallow_ strings without considering
the case. I found in the Net same robots.txt files with the disallow directive
written lowercase and so not considered by links.

Than other public Apis are just a wrapper to this private method making the dirty work.

```ruby Links::Api.get - get the page
def self.get(url)
  begin
    uri = URI(url)
    if uri.scheme == 'http'
      res = Net::HTTP.get_response(URI(url))
    else
      request=Net::HTTP.new(uri.host, uri.port)
      request.use_ssl=true
      request.verify_mode = OpenSSL::SSL::VERIFY_NONE
      res = request.get(uri.request_uri)
    end
    return res
  rescue
    return nil
  end
end
``` 

links rubygem however has another _secret_ goal that I'm working on: being a
full featured site crawler. The Api revealing that is the Links::Api.links.

``` ruby Links::Api.links - get all links in a webpage
def self.links(url)
  res = Links::Api.get(url)
  if res.nil?
    return []
  end
  doc = Nokogiri::HTML.parse(res.body)
  l = doc.css('a').map { |link| link['href'] }
  l
end
``` 

## What we've learnt?

This hack for fun post was about how gather information using robots.txt file,
as developer you have no power but you can enforce the server hardening by
removing all unnecessary files and directories.

1. attackers can use robots.txt file to discover our website sections that are
   supposed to stay private
2. we must carefully check if a private webpage has been requested for a non
   authenticated session
3. you can code a piece of ruby code just for fun :-)
