---
layout: post
title: "Getting root: Matrix"
image: assets/images/matrix.jpg
bootstrap: true
tags: [boot2root, vulnhub, getting-root]
categories: [ getting-root ]
author: thesp0nge
---

It was last year when I received the email saying I passed the [Penetration
testing with Kali
Linux](https://www.offensive-security.com/information-security-training/penetration-testing-training-kali-linux/)
course and eventually I became an OSCP guy.

Now, an year later, I just finished the material for the [Cracking the
Perimeter](https://www.offensive-security.com/information-security-training/cracking-the-perimeter/)
course and later this spring I will try to obtain the OSCE certification.

Today I played with the [Matrix Vulnhub
machine](https://www.vulnhub.com/entry/matrix-1,259/), a very easy boot2root
image you can start with if you're starting studying offensive security
techniques.

## First stage: reconnaissance

Our boot2root machine is available on a internal host only virtuablbox network
and the IP address is 192.168.56.104.

As all takeover journeys, the first step is to nmap the target discovering
opened ports and associated services.


As we can see, we've got SSH server running on port 22 and two webservers on
port 80 and 31337.
We can use _dirb_ tool to enumerate website content, discovering pages and
other resources.


From the video below, you can see that the intended path was hidden in HTML source.
On webserver port 80, the "service" hint was a rabbit picture and the filename
suggests to see at port 31337.

On port 31337, there is a base64 encoded hint.

{% include youtube.html id="gD_w0sbGFZg" %}

Decoding the base64, we can see as a suggestion this statement:

{% highlight sh %}
echo "Then you'll see, that it is not the spoon that bends, it is only yourself. " > Cypher.matrix#
{%endhighlight%}


## Let's go and break the door

This hint suggests a Cypher.matrix file is present on the second webserver.
Let's try to download it.


It's a [brainfuck](https://en.wikipedia.org/wiki/Brainfuck) script, we can
decode it with _beef_ tool. You can install it, on a Kali Linux with _apt
install beef_.


The script content is another hint, about the guest account ssh password.

{% highlight sh %}
You can enter into matrix as guest, with password k1ll0rXX
Note: Actually, I forget last two characters so I have replaced with XX try your luck and find correct string of password.
{%endhighlight%}

We will use _crunch_ tool to generate an 8 characters password starting with
_k1ll0r_ with the latest two characters substituted with all possible
alphanumeric combinations.


We will use ncrack, passing the generated list, to find the password.

The guest account ssh password is k1ll0r7n.

## Root dance

Now we can log into the machine as _guest_ user. As you can see from the video,
_guest_ account enters in a restricted shell environment; path is limited and
we can't launch ls to go deeper into our journey. Luckly enough _vi_ editor is
in our PATH, we can then use the _\!/bin/bash_ escape sequence to have a shell
spawned for us.

This shell is not restricted, event the PATH variable is still mangled.
Starting the post exploitation steps, a misconfiguration in sudoers file
reveals that every account on the system can launch sudo to launch every
command as root.

So we will use the _guest_ password to launch a shell as root. Since we do want
to have a working environment, we will ask sudo to launch a login shell (bash
-l) to have the configuration files read and everything setted up.


## Off by one

This machine is very simple but it can teach a couple of interesting lessons:

* look at web page source code;
* brainfuck programs are not a problem with _beef_ tool;
* before looking to exploits, start post exploitation looking at the system
  configuration. Sometimes, it's the human error the simplest path to root.

Enjoy it!
