---
layout: post
title: "How to calculate your network address in CIDR notation"
image: assets/images/network.jpg
bootstrap: true
tags: [bash, ip, netmask, cidr, script]
categories: [ armoredbasics ]
author: thesp0nge
---

Sometime I need to quick nmap the network just right cable plug. Since I'm lazy
I created a simple bash script to calculate the network address in CIDR
notation, starting from ifconfig output.

The code is available
[here](https://gist.github.com/thesp0nge/72c91cd96e4916914548f329fd787a0c) with
a MIT LICENSE.

I started from [this gist](https://gist.github.com/hmmbug/e386274e299ab4d00a9f)
I found on Github. On my virtual machine, the ```hostname -i``` command doesn't
work. The machine resolves itself as localhost and the IP address returned is
127.0.0.1.
I used ```ifconfig``` instead, even if this command is deprecated. Please,
meanwhile I will update the script to use ```ip``` command, install net-tools
package.

We have also to support hosts with more than a network interface, so our script
will allow a parameter to passed to command line to specify the network card we
want to calculate the address for.

In order to calculate the "/xx" suffix I used the
[this](https://stackoverflow.com/questions/50413579/bash-convert-netmask-in-cidr-notation)
IPprefix_by_netmask implementation in bash I found on StackOverflow.

After some bash-fu, error checking routines and parameter support, here it is
the final version of the script:

{% gist 72c91cd96e4916914548f329fd787a0c %}


Now I can easily use this output to run nmap on every network I will be
attached to... and I improved a bit my bash scripting skills.

Enjoy it!
