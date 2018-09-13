---
layout: post
title: "Railsberry chronicles: day 2 - The English penetration test (eventually the day I talk to 450+ oustanding developers)"
date: 2013-04-23 14:50
comments: true
published: true
featured: false
categories: ruby conference talk railsberry poland cracow 
thumb: thesp0nge-talk.png
level:
hn: 
rd: 
---

Finally the day I gave the talk is arrived and it's gone. Going on stage in
front a more than 450 talented developers was an astonishing experience. It
drove me crazy. My spoken English has limits on its own, but it in front of
such crowd I seemed to be a scared 4 years old child.

However, talk was good afterall. Everything went well. Nothing broke during
exposure, none of the people were harmed during the talk, no customer ewb
applications were broken Internet is still working ( I guess ).

<!-- more -->

## A particular mention to... 

[Felix Geisendoerfer](http://www.railsberry.com/speakers#felix) gave us today
an **oustanding** talk about to make an [http://nodecopter.com/](drone) to fly
controlled by javascript or any other programming language.

Kudos to [https://twitter.com/felixge](Felix) for his hacks and for great talk.

## My slides and the videos

The code you need to play against a web application is:

``` 
$ gem install ciphersurfer
$ gem install gengiscan
$ gem install codesake_links
$ gem install cross
``` 

Soon [cross](https://github.com/thesp0nge/cross),
[gengiscan](https://github.com/thesp0nge/gengiscan) and
[ciphersurfer](https://github.com/thesp0nge/ciphersurfer) will but under the
[codesake](https://github.com/codesake) project and eventually it will
integrate into the dusk tool (repository is not created).

[dawn](https://github.com/codesake/codesake_dawn) code review tool is going to
be soon updated with testing for
[CVE-2013-1800](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2013-1800).

The idea is that both dusk and
[dawn](https://github.com/codesake/codesake_dawn) will be the core engines
behind [codesake.com](http://codesake.com) application security startup, but
it's quite early to talk about it. The thing to remember is that the security
engines will be opensource, **ever**.

So, I hope you enjoyed the talk. In case you missed, because you were not there, here is my slides:

{% speakerdeck 6eedd1508e3f01305f0312313815291c %}

With demo videos too.

### Railsberry 2013 - Navigating the attack target after the information gathering stage
{% youtube NgNhp_bHsgM %} 

### Railsberry 2013 - First XSS spotted in the wild
{% youtube DyyfdUUf9-4 %}  

### Railsberry 2013 - Information gathering
{% youtube TVQGep-kXwQ %} 

### Railsberry 2013 - Bruteforce users login name
{% youtube -iLoTW26SKg %} 

### Railsberry 2013 - Find reflected XSS with cross
{% youtube -3I185hGjCQ %} 

If you have any questions, comments or criticisms please use my [ask me anything box](https://github.com/armoredcode/feedback/issues) on
[github](https://gitub.com).

Enjoy it!
