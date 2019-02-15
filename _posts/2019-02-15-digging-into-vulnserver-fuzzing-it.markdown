---
layout: post
title: "Digging into Vulnserver: fuzzing it"
image: assets/images/hole.jpg
bootstrap: true
tags: [vulnserver, osce, fuzzing, exploit development]
categories: [ osce, vulnserver ]
author: thesp0nge
---

[Vulnserver](https://github.com/stephenbradshaw/vulnserver) is a Win32
application built to simulate a TCP/IP server listening on port 9999 and
accepting commands from unauthenticated clients.

Some commands are vulnerable to different kinds of [buffer
overflow](http://phrack.org/issues/49/14.html#article), some other commands are
not vulnerable at all.

The goal of this software is to train people into exploit development under
some very particular situations.

## My setup

As you might recall from [a last month
post]({{site.url}}/blog/a-cracking-the-perimeter-journey-1-my-own-lab/), I
created a Windows 7 virtual machine for the Cracking the perimeter course
exploit development training.

Since this journey into Vulnserver.exe pants is tight with the upcoming exam, I
decided to write exploit on this environment. 

![The vulnserver daemon listening]({{site.url}}/assets/images/vulnserver/clean_windows.png)

For sake of semplicity I disabled both ASLR and DEP protection mechanisms using
[Microsoft
EMET tool.](https://support.microsoft.com/it-it/help/2458544/the-enhanced-mitigation-experience-toolkit)

![ASLR and DEP protections disabled]({{site.url}}/assets/images/vulnserver/disabled_dep_aslr.png)

Vulnserver is now listening on TCP port 9999 waiting for (evil) client
connections.

## The fuzzer

Befor even start coding a single exploit we must understand which are the
vulnerable commands. Connecting to the server issuing the 'HELP' command we can
have the full list of available commands.

![The commands list]({{site.url}}/assets/images/vulnserver/telnet.png)

The basic idea is to create a [SPIKE fuzzer
template](https://resources.infosecinstitute.com/intro-to-fuzzing/#gref) to
each command and start feeding the server with buffers full of 'A' trying to
triggering some error.

However, there are too many commands with too many templates to write. So, why
don't writing my own specific fuzzer, working on every single command?

Here it is.

{% gist b75654e452fcf70025f1a4e56c322526 %}

As you can see, available commands are defined as array of strings. The
```is_vulnerable_command``` routine is responsible by feeding the command
with tons of 'A' characters and waiting for some networking error. If a
communication error occurs, the fuzzer thinks vulnerable server eventually goes
down and then it returns the offending buffer lenght.

If nothing happens, the command is marked to be safe.

In the main loop, in order to handle the program restarts, I added a "wait for
keypress" just continuing the loop after the vulnserver.exe it has been
restarted.

{% asciicast 227831 %}

As you can see, when the fuzzer asks for a key press when it receives a socket
error. When there is a socket error, on the windows machine, the situation is
the one shown into this picture.

![The crash]({{site.url}}/assets/images/vulnserver/crash.png)

So everything is ok? Unfortunately not. I'm quite satisfied about my python
script but, since solutions are public, there are two commands (TRUN and GMON)
that were not detected.

So I can improve my fuzzer? Luckly for us, yes we can improve it.
The reason is that those two commands vulnerabilities were triggered by
unexpected characters so that an exception is triggered. To cause this, I
change the way my fuzzer build the evil payload.

Instead of sending only 'A's, I'll send also digits and special characters
hoping an exception it will be triggered.

{% highlight python %}
payload = command + " " + ''.join(random.choice(string.ascii_uppercase + string.digits + string.punctuation) for _ in range(i))
{% endhighlight %}

As you can see, a buffer built this way is able to trigger the buffer even on
TRUN command.

![trun]({{site.url}}/assets/images/vulnserver/trun.png)

Good.

Now I will play again a full command fuzzing and hopefully I have the full list
of vulnerable commands and the offending buffer length.

{% asciicast 227838 %}

Now, with the list of vulnerable commands in my hand, I can start my exploit
development journey.

{% highlight sh %}
[*]  KSTAN  is safe
[+]  TRUN  -  2100
[+]  GMON  -  4000
[+]  KSTET  -  100
[+]  GTER  -  200
[+]  HTER  -  4200
[+]  LTER  -  2100
{% endhighlight %}

Enjoy it!
