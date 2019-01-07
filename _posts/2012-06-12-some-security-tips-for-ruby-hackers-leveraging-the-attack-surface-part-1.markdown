---
layout: post
title: "Some security tips for ruby hackers: leveraging the attack surface. Part 1."
date: 2012-06-13 17:25
comments: true
published: true
featured: false
tags: builders breakers ruby webapp gems pentest speech rubyday owasp owasp-testing-guide pentest-with-ruby
hn: http://news.ycombinator.com/item?id=4202206
rd: http://redd.it/w2i0s
---

In the [first episode](/blog/some-security-tips-for-ruby-hackers-prelude/) I
introduced the security checks I'd like to talk about at the
[talk](http://rubyday.it/talks/2/) I have to give next Friday.

Today we will talk about the code to automate this checks.

<!-- more -->

## The attack surface

Discovering the attack surface it will be the first part of my
[talk](http://rubyday.it/talks/2/). It's about:

<table class="table-striped table-bordered table">
  <thead>
    <tr>
      <th>Category</th>
      <th>Owasp Testing guide reference</th>
      <th>Test name</th>
    </tr>
  </thead>
  <tbody>
    <tr> <td>Information Gathering</td> <td>OWASP-IG-001</td> <td>Spiders, Robots and Crawlers</td> </tr>
    <tr> <td>Information Gathering</td> <td>OWASP-IG-002</td> <td>Search Engine Discovery/Reconnaissance</td> </tr>
    <tr> <td>Information Gathering</td> <td>OWASP-IG-003</td> <td>Identify application entry points</td></tr>
    <tr> <td>Information Gathering</td> <td>OWASP-IG-004</td> <td>Testing for Web Application Fingerprint</td></tr>
    <tr> <td>Information Gathering</td> <td>OWASP-IG-005</td> <td>Application Discovery</td></tr>
    <tr> <td>Information Gathering</td> <td>OWASP-IG-006</td> <td>Analysis of Error Codes</td></tr>
    <tr> <td>Configuration Management Testing</td> <td>OWASP-CM-001</td> <td>SSL/TLS Testing</td></tr>
    <tr> <td>Configuration Management Testing</td> <td>OWASP-CM-002</td> <td>DB Listener Testing</td></tr>
    <tr> <td>Configuration Management Testing</td> <td>OWASP-CM-003</td> <td>Infrastructure Configuration Management Testing</td></tr>
    <tr> <td>Configuration Management Testing</td> <td>OWASP-CM-004</td> <td>Application Configuration Management Testing</td></tr>
    <tr> <td>Configuration Management Testing</td> <td>OWASP-CM-005</td> <td>Testing for File Extensions Handling</td></tr>
    <tr> <td>Configuration Management Testing</td> <td>OWASP-CM-006</td> <td>Old, backup and unreferenced files</td></tr>
    <tr> <td>Configuration Management Testing</td> <td>OWASP-CM-007</td> <td>Infrastructure and Application Admin Interfaces</td></tr>
    <tr> <td>Configuration Management Testing</td> <td>OWASP-CM-008</td> <td>Testing for HTTP Methods and XST</td></tr>
  </tbody>
</table>

Let's go.

## OWASP-IG-001: Spiders, Robots and Crawlers

In April I wrote a post about [using robots.txt as attack weapon](http://armoredcode.com/blog/h4f-use-robots-dot-txt-as-a-weapon-with-links-rubygem/).
Do you remember it?  No?!? Go back and [read it](http://armoredcode.com/blog/h4f-use-robots-dot-txt-as-a-weapon-with-links-rubygem/).

Acting as a spider it is possible to discover how much your website is wide and
to spot interesting entry points.

Robots.txt file is the first thing I test if I want to find more out of your
site.

[links](https://github.com/thesp0nge/links) rubygem is a piece of code I wrote
to automate OWASP-IG-001 testing.

As you may see... the Net::HTTP is enough to play with this test.

``` ruby Links::Api.robots - testing for OWASP-IG-001
# TESTING: SPIDERS, ROBOTS, AND CRAWLERS (OWASP-IG-001)
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
        if (line.downcase.start_with?('disallow'))
          list << line.split(":")[1].strip.chomp
        end
      else
        if (line.downcase.start_with?('allow') or line.downcase.start_with?('disallow'))
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
In the bin/links ruby script we check if the link disallowed is accessible or not.
Discovering disallowed urls that are accessible is **important** if we're
wondering to discover service door and try to break-in

``` ruby what we can do with robots.txt content (again from links rubygem)
list.each do |l|
  if robots or bulk
    if ! l.start_with? '/'
      l = '/'+l.chomp
    end
    if ! target.start_with? 'http://' and ! target.start_with? 'https://'
      #defaulting to HTTP when no protocol has been supplied
      target = "http://"+target
    end

    print "#{target}#{l}:".color(:white)
    start = Time.now
    code = Links::Api.code(target+l, proxy)
    stop = Time.now
  else
    print "#{l}:".color(:white)
    start = Time.now
    code = Links::Api.code(l, proxy)
    stop = Time.now
  end
...
```

### Crawling: the clean way

What about crawling a website? By crawling I mean retrieving all the possible
urls starting from the homepage, extracting all the links in the HTML and
recursive make a lot of requests.

But we're lucky enough and there is [something who make a great gem for us](http://anemone.rubyforge.org/).

Using anemone rubygem, we have a clean DSL for crawling a website starting from
the links extracted by the web pages we find.

``` ruby crawling a website using anemone 
require 'anemone'

Anemone.crawl("http://www.target.com/") do |anemone|
  anemone.on_every_page do |page|
      puts page.url
  end
end
``` 

### Crawling: the bruteforce way

Even before discovering [anemone](http://anemone.rubyforge.org/) rubygem, I
wrote the [enchant](https://github.com/thesp0nge/enchant) gem to discover links
by bruteforcing the url with words taken from dictionary.

Using a bruteforce approach can be useful if an important link is not in
robots.txt (and I do suggest not to do this) and it's likely not linked in any
of the public pages.

Enchant::Engine.get_list method is trivial, it take the words from a dictionary I borrow from [Owasp Zap](https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project) project.

``` ruby Enchant::Engine.get_list
def get_list

  if @wordlist.nil?
    if File.exists?('../../db/directory-list-2.3-small.txt')
      @wordlist='../../db/directory-list-2.3-small.txt'
    end
    if File.exists?('./db/directory-list-2.3-small.txt')
      @wordlist='./db/directory-list-2.3-small.txt'
    else
      @list = {}
    end

  end

  begin
    File.open(@wordlist, 'r') { |f|
      @list = f.readlines
    }
  rescue Errno::ENOENT
    puts "it seems the wordlist file is not present (#{@wordlist})".color(:red)
    @list = {}
  end
end

``` 

There is no real magic in the Enchant::Engine.scan method... just a bunch of get and check for error codes... I know, I won't win the A.Turing awards for these pieces of code, but sometimes they saved me the day in real pentest.

``` ruby Enchant::Engine.scan main loop
list.each do |path|
  pbar.inc
  if ! path.start_with? '#'
    begin
      response = http.get('/'+path.chop)
      c = response.code.to_i
      refused = 0
      if c == 200
        @urls_open << path
      end
      if c == 401
        @urls_private << path
      end
      if c >= 500
        @urls_internal_error << path
      end
    rescue  Errno::ECONNREFUSED
      refused += 1
      if refused > 5
        pbar.finish
        puts "received 5 connection refused. #{@host} went down".color(:red)
        return @urls_open.count 
      else
        puts "[WARNING] connection refused".color(:yellow)
        sleep 2 * refused
      end
      
    rescue Net::HTTPBadResponse
      refused = 0
      if @verbose
        puts "#{$!}".color(:red)
      end
    rescue Errno::ETIMEDOUT
      refused = 0
      if @verbose
        puts "#{$!}".color(:red)
      end
    end
  end
end
```

## OWASP-IG-002: Search Engine Discovery/Reconnaissance

This task can be done easily with a browser. Just point it to
[google.com](http://google.com) and use the 'site:' special keyword to search
for all pages about a website indexed with google.

A sample query that enumerate all the stuff you can find related to
armoredcode.com domain is:
[http://www.google.it/search?q=site:armoredcode.com](http://www.google.it/search?q=site:armoredcode.com)

Of course you can use Net::HTTP also in this case, but Google is not happy to
be called in an automated way without authentication and their api usage... so
it's easy not to automate the task at all :-)


## OWASP-IG-004: Testing for Web Application Fingerprint

This is a 2 years old project, may be it would a great idea to write down a new
and better fingerprinter, however
[wafp](http://code.google.com/p/webapplicationfingerprinter/) script can be
used to try to detect the CMS version or a particular Application server
serving our target.

## OWASP-CM-001: SSL/TLS Testing

For SSL/TSL testing I use a rubygem I wrote a couple of months ago: [ciphersurfer](https://github.com/thesp0nge/ciphersurfer).

I blogged about ciphersurfer in my previous blog:
[here](http://thesp0nge.com/blog/2012/01/25/is-my-connection-really-secure/)
and
[here](http://thesp0nge.com/blog/2012/02/01/announce-ciphersurfer-v1-dot-0-0/).

Maybe those two posts deserve a repost over armoredcode.com.

However the trick behind ciphersurfer is trying to make HTTPS calls, using
standard Ruby networking APIs (against, no voodoo here).

``` ruby lib/ciphersurfer/scanner.rb
def go
  context=OpenSSL::SSL::SSLContext.new(@proto)
  cipher_set = context.ciphers
  cipher_set.each do |cipher_name, cipher_version, bits, algorithm_bits|

    request = Net::HTTP.new(@host, @port)
    request.use_ssl = true
    request.verify_mode = OpenSSL::SSL::VERIFY_NONE
    request.ciphers= cipher_name
    begin
      response = request.get("/")
      @ok_bits << bits
      @ok_ciphers << cipher_name
    rescue OpenSSL::SSL::SSLError => e
      # Quietly discard SSLErrors, really I don't care if the cipher has
      # not been accepted
    rescue 
      # Quietly discard all other errors... you must perform all error
      # chekcs in the calling program
    end
  end
end
```

Here we don't use httpclient helpers since I want to play with different
ciphers at time.

That's it. All the magic happens there. Now, let's look like at the bin script
to see how the scoring system has been used.

First of all, we must scan the target for all the protocols we support.

``` ruby bin/ciphersurfer
protocol_version.each do |version|
  s = Ciphersurfer::Scanner.new({:host=>host, :port=>port, :proto=>version})

  s.go
  if (s.ok_ciphers.size != 0)
    supported_protocols << version
    cipher_bits = cipher_bits | s.ok_bits
    ciphers = ciphers | s.ok_ciphers
  end

end
```


``` ruby bin/ciphersurfer
cert= Ciphersurfer::Scanner.cert(host, port)
if ! cert.nil?
  a=cert.public_key.to_text
  key_size=/Modulus \((\d+)/i.match(a)[1]
else
  puts "warning: the server didn't give us the certificate".color(:yellow)
  key_size=0
end
```

Note that we don't make another GET here since we did it at the beginning of
the engagement when we checked if the target was alive or not.

Now, let's calculate the scores, all of them in a 0..100 range.

``` ruby bin/ciphersurfer
proto_score=  Ciphersurfer::Score.evaluate_protocols(supported_protocols)
cipher_score= Ciphersurfer::Score.evaluate_ciphers(cipher_bits)
key_score=    Ciphersurfer::Score.evaluate_key(key_size.to_i)
score=        Ciphersurfer::Score.score(proto_score, key_score, cipher_score)
```

And then, some graphics to make the experience more appealing.

``` ruby bin/ciphersurfer
printf "%20s : %s (%s)\n", "Overall evaluation", Ciphersurfer::Score.evaluate(score), score.to_s
printf "%20s : ", "Protocol support" 
proto_score.to_i.times{print 'o'.color(score_to_color(proto_score))}
puts ' ('+proto_score.to_s+')'
printf "%20s : ",  "Key exchange" 
key_score.to_i.times{print 'o'.color(score_to_color(key_score))}
puts ' ('+key_score.to_s+')'
printf "%20s : ", "Cipher strength" 
cipher_score.to_i.times{print 'o'.color(score_to_color(cipher_score))}
puts ' ('+cipher_score.to_s+')'
```

## Wrap up

This is the first episode about leveraging the attack surface of a web application and, as along I was writing it I realized a couple of things:

* in the friday talk I'll go completely out of time
* there is a lot of things to say about using ruby and the [Owasp testing guide](https://www.owasp.org/index.php/OWASP_Testing_Guide_v3_Table_of_Contents)
  that it's worth making something bigger...
* I have a lot of things yet to learn
* I don't have fancy pictures to put on my Friday slideshow

## Reference

Please note that this post series is built using [Owasp testing guide](https://www.owasp.org/index.php/OWASP_Testing_Guide_v3_Table_of_Contents)
as skeleton.

## Off topic

True to be told I'm nervous for #rubyday talk. The talks I gave since today
were done at security conferences where I'm confortable. 

Here I'm going to talk about code to people who **do** uunderstand and that
they **do** write great code, and they are not afraid to show it.

Just a bit scared... I hope they like it.
