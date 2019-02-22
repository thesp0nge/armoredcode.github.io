---
layout: post
title: "A closer look to msf-egghunter"
image: assets/images/eggs.jpg
bootstrap: true
tags: [exploit development, metasploit framework, egg hunter]
categories: [ ]
author: thesp0nge
---

The egghunting is a technique used in exploit writing to deal with evil
shellcode to be placed in a memory location different from the one we are
redirected via EIP overwrite or SEH hijack or other.

The basic idea behind egg hunter is that shellcode is prepended by a 4 bytes
long pattern which is repeated twice and a small routine will search into
memory for this pattern. When the pattern it has been found, the routine will
jump on that memory location that eventually contains the shellcode we want to
execute.

Egg hunting technique is introduced in [Safetly Searching Process Virtual
Address Space](http://www.hick.org/code/skape/papers/egghunt-shellcode.pdf)
paper by Matt Miller.

I wrote about egg hunting during [my SLAE
journey](https://codiceinsicuro.it/slae/assignment-3-an-egg-hunter-journey/)
back in May 2018.

Today I want to introduce you the
[msf-egghunter](https://github.com/rapid7/metasploit-framework/blob/master/tools/exploit/egghunter.rb)
tool by Metasploit Framework.

Instead of writing your egghunting routine from scratch (hint: do it at least
one time in order to understand how it works), you have a ready to be used
helper with your Metasploit installation.

{%highlight sh%}
[root:~/src/hacking/toolbox/utils]# msf-egghunter -h
Usage: msf-egghunter [options]
Example: msf-egghunter -f python -e W00T

Specific options:
    -f, --format <String>            See --list-formats for a list of supported output formats
    -b, --badchars <String>          (Optional) Bad characters to avoid for the egg
    -e, --egg <String>               The egg (Please give 4 bytes)
    -p, --platform <String>          (Optional) Platform
        --startreg <String>          (Optional) The starting register
        --forward                    (Optional) To search forward
        --depreg <String>            (Optional) The DEP register
        --depdest <String>           (Optional) The DEP destination
        --depsize <Integer>          (Optional) The DEP size
        --depmethod <String>         (Optional) The DEP method to use (virtualprotect/virtualalloc/copy/copy_size)
    -a, --arch <String>              (Optional) Architecture
        --list-formats               List all supported output formats
    -v, --var-name <name>            (Optional) Specify a custom variable name to use for certain output formats
    -h, --help                       Show this message

{%endhighlight%}

The basic usage here is to specify the platform (windows or linux) and the
architecture since the tool will produce different assembler code. The only
mandatory parameter is the egg to be used, BEEF in our case.

{% highlight sh %}
[root:~/src/hacking/toolbox/utils]# msf-egghunter -p windows -a x86 -f python -e BEEF
buf =  ""
buf += "\x66\x81\xca\xff\x0f\x42\x52\x6a\x02\x58\xcd\x2e\x3c"
buf += "\x05\x5a\x74\xef\xb8\x42\x45\x45\x46\x89\xd7\xaf\x75"
buf += "\xea\xaf\x75\xe7\xff\xe7"

[root:~/src/hacking/toolbox/utils]# msf-egghunter -p linux -a x86 -f python -e BEEF
buf =  ""
buf += "\xfc\x66\x81\xc9\xff\x0f\x41\x6a\x43\x58\xcd\x80\x3c"
buf += "\xf2\x74\xf1\xb8\x42\x45\x45\x46\x89\xcf\xaf\x75\xec"
buf += "\xaf\x75\xe9\xff\xe7"
{% endhighlight %}

What about bad characters than? Accordingly to the tool help message, with the
'-b' parameter you supply the characters to be avoided... but it doesn't work.

My Metasploit version is 5.0.6-dev and if I need to avoid '\x66' character in
the latest example I have some trouble.

{%highlight sh%}
[root:~/src/hacking/toolbox/utils]# msf-egghunter -p linux -a x86 -f python -e BEEF -b '\x66'
buf =  ""
buf += "\xfc\x66\x81\xc9\xff\x0f\x41\x6a\x43\x58\xcd\x80\x3c"
buf += "\xf2\x74\xf1\xb8\x42\x45\x45\x46\x89\xcf\xaf\x75\xec"
buf += "\xaf\x75\xe9\xff\xe7"
{%endhighlight%}

The usage combo I love most is the one with msfvenom. msfvenom is not good only
to create shellcode, instead it could also read a code from standard input and
apply some filters over it.

If we want to avoid the '\x66' character in our egg hunter code, the best way
to achieve it is to ask msfvenom to apply some encoding. Let's do this way:

{%highlight sh%}
[root:~/src/hacking/toolbox/utils]# msf-egghunter -p linux -a x86 -e BEEF -f raw | msfvenom -p - --platform linux -a x86 -b '\x66' -f python

Attempting to read payload from STDIN...
Found 11 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 58 (iteration=0)
x86/shikata_ga_nai chosen with final size 58
Payload size: 58 bytes
Final size of python file: 292 bytes
buf =  ""
buf += "\xd9\xe9\xbd\x34\x49\xad\xb8\xd9\x74\x24\xf4\x5e\x29"
buf += "\xc9\xb1\x08\x83\xc6\x04\x31\x6e\x16\x03\x6e\x16\xe2"
buf += "\xc1\xb5\xcb\x39\xe0\xb9\x1b\x7b\x98\x06\x7c\xb6\xdc"
buf += "\xb5\x8e\x3c\x2d\x7d\x2c\xf8\x88\x38\x38\xcd\xbd\xb0"
buf += "\xd6\x7d\xb4\xd3\xd8\x65"

{%endhighlight%}

We asked msf-egghunter to output a raw output we pass to msfvenom specifying
the list of bad characters. After the encoder pass, we can see there is no
'\x66' character in out code.

The msf-egghunter core is in the rex-exploit library by Rapid7, you can find
the source code on
[GitHub](https://github.com/rapid7/rex-exploitation/blob/master/lib/rex/exploitation/egghunter.rb).

Looking at the very beginning of the source code, we can read in the comments
the improvements made starting from the Matt Miller's work.
Very clever code is the code to [disable
DEP](https://www.corelan.be/index.php/2010/06/16/exploit-writing-tutorial-part-10-chaining-dep-with-rop-the-rubikstm-cube/)
for Windows platform made by corelandc0d3r

## Off by one

Egg hunting is a very fun technique to be used in exploit development. You tell
the machine to search the memory for the code it must use to compromise itself.

The best combination is with msfvenom to create fancy encoded version of your
egg hunter.

And you? Which is your egg hunting usage when writing exploits? Do you write
custom code or do you write some other tool?

