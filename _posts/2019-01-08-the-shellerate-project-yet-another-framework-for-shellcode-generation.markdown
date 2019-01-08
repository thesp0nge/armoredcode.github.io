---
layout: post
title: "The shellerate project: yet another framework for shellcode generation"
image: assets/images/shells.jpg
bootstrap: true
tags: [python, tool, shellcode, slae, polymorphism, encoding, x86]
categories: [ tools, offensive ]
author: thesp0nge
---

Last summer, as I told on [Codice Insicuro](https://codiceinsicuro.it/slae/),
my Italian blog about cybersecurity and related, I took the [x86 Assembly
Language and Shellcoding on
Linux](https://www.pentesteracademy.com/course?id=3) course and related
certification.

The idea was to train myself into advanced shellcode programming and anti-virus
evasion challenges I will face now in the [Cracking the
perimeter](https://www.offensive-security.com/information-security-training/cracking-the-perimeter/)
course.

From the assignments I did, in order to obtain the SLAE32 certification I
created the [shellerate](https://github.com/thesp0nge/shellerate) project. The
shellerate term is a pun between 'shell' and 'scellerato', the Italian word for
'wicked', so sorry for English mother tongue people out there, to have invented
a new 'world' in yout language. The ideal pronunciation is the union of "shell"
and "rate" words.

Back to the techie part, shellerate is a shellcode generation framework. There
are tons of excellent frameworks out there, I started a new project in order to
learn python better and to have a DIY tool in order to customize my exploits.

[shellerate](https://pypi.org/project/shellerate/) is a standard project, you
can install using pip.

{% highlight sh %}
$ pip3 install shellerate --user
{% endhighlight %}

Installing shellerate, 0.3.0 version is the latest as the time I'm writing
this, you will install a very alpha shellcode generation framework that makes
you able to create a bind shell shellcode for x86 platform and Linux operating
system only. 

You can
[encode](https://codiceinsicuro.it/slae/assignment-4-a-default-encoder/) the
shellcode with the custom encoding schema I developed.

{%highlight python%}
from shellerate.bind_shellcode import *

b=BindShellcode(4444, 'x86', 'linux')
b.encode()
b.generate()
print("Shellcode: %s" % b.shellcode())

{%endhighlight%}

As you can seem I'm trying to create a clean API around the type of shellcode
you want to create:

* a TCP bind shell
* a TCP reverse shell

You can also create an [egg hunter
version](https://codiceinsicuro.it/slae/assignment-3-an-egg-hunter-journey/) in
a very similiar way:

{%highlight python%}
b=BindShellcode(4444, 'x86', 'linux')
b.egg_hunter()
b.generate()
sc = b.shellcode()

print("Egg Hunter: %s" % sc["egg_hunter_code"])
print("Shellcode: %s" % sc["egg_hunter_shellcode"])
{%endhighlight%}

## Next steps

Tons of future implementations are in my personal roadmap. First of all, I have
to add a [polymorphic
engine](https://codiceinsicuro.it/slae/assignment-6-generate-a-polymorphic-shellcode/)
and to replicate all features on the reverse shell shellcode generator.

Then I have to extend the support to other operating systems and to x86-64
architecture.

Since must of this is for my OSCE exam, I think that win32 platform will be the
next I will support in the very next months.

## Off by one

I don't know I anyone of you reading this post, will found shellerate useful. I
created it just for fun and I had a lot of it during those months of work. I
hope the funny part will last for long time.

Please feel free to spread the voice about the project and if you have
suggestions ot criticisms about shellerate, please comment this post and share
with me your opinions.

Enjoy it!
