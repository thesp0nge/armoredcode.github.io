---
layout: post
title: "Enabling related post on octopress and Mac OS X"
date: 2012-08-29 09:27
comments: true
published: true
featured: false
tags: octopress blog gsl configuration howto related-post
hn: 
rd: http://redd.it/z0cl6
---

[Octopress](http://octopress.org) is a powerful framework built over [Jekyll](https://github.com/mojombo/jekyll) to create static websites.
I used [Octopress](http://octopress.org) too for
[armoredcode.com](http://armoredcode.com). All posts are written with
[vim](http://www.vim.org) using
[markdown](http://daringfireball.net/projects/markdown/) with some javascript
to integrate with [disqus for comments](http://disqus.com/),
[github](http://www.github.com) or [twitter](http://www.twitter.com).

No database, just a webserver and a bunch of html files you're reading right now. 

Yesterday I turned on option to show related posts. Let's see how.

<!-- more -->

Yesterday while I was editing octopress wiki page to add [highlight plugin I wrote](https://github.com/thesp0nge/octopress_highlight_plugin) in the list I
found something I was looking for from months: a plugin for [related posts](https://github.com/jcftang/octopress-relatedposts).

Showing related posts is one of the key features I was missing more after I
switched from [wordpress](http://www.wordpress.org) to statically generated
websites.

Looking at the plugin's [README](https://github.com/jcftang/octopress-relatedposts/blob/master/README.md) I was surprised in finding that related posts is a builtin feature wordpress already have. To enable post classification you have just to turn the lsi parameter on in your _config.yml.

``` 
...
lsi: true
...
``` 

However internal engine to classify posts is very slow and octopress itself
suggests you to install [Ruby/GSL bindings](http://rb-gsl.rubyforge.org/) to
speed up the process.

## Installing GNU Scientific Library

Before installing the ruby binding you have to install the GNU Scientific
Library that it will be used to classify posts. 

If you're using [brew](http://mxcl.github.com/homebrew/) as package manager,
and you really should do, installing the library it's just a command away:

``` 
$ brew install gsl
``` 

On my Mac OS X 10.6, brew complains that some directory wasn't writable so
you've to manually chmod them giving your user a temporary writing access. Make
sure to revert the permission schema in order not to make your directory
permissions too loose.

If you're not using brew you can follow the Unix way to install
GSL: download the tarball from the
[website](http://www.gnu.org/software/gsl/#downloading), unpack it, launch
configure then make then make install as root.

For other Mac OS X package managers I don't know if they can install GSL, maybe
you can try on your own.

## Installing the Ruby/GSL binging

To let this step to pass, make sure the gsl-config executable is in your $PATH.
Important note: highlight if you're using [rvm](https://rvm.io/) make sure to
switch to the ruby version and eventually to the gemset you're using with
octopress or it won't find GSL when running the classifier.

Before installing Ruby/GSL, if you're under Mac OS X you have to apply [this patch](https://gist.github.com/1217974)
to the source code. Just download and use the venerable patch command:

```
$ patch -p1 < ~/Downloads/gist1217974-b9f7ea8c9372d321c81fb8c12a272d76ecd9f1ef/rb-gsl.patch
``` 

Patch applies smoothly and you can proceed configuring the package:

```
$ ruby setup.rb config
```

Setting it up and compiling with:

```
$ ruby setup.rb setup 
```

My system has this gcc version, it produces some warning while compiling that you can safetly ignore:

``` 
$ gcc -version                                                                             
i686-apple-darwin11-llvm-gcc-4.2: no input files
```

Last step is to install Ruby/GSL binding, you must to be root for this:

``` 
$ sudo ruby setup.rb install
``` 

## Show your related posts

Now everything should work and every time you run a generate command from your
octopress root, Ruby/GSL should be used as classifier engine and posts
correlation should be very fast.

Now it's time to show your related posts.

Since [this plugin](https://github.com/jcftang/octopress-relatedposts) makes me
to discover the related post feature in octopress I really encourage you to use
it. 

For my own taste, I don't like to show related post in an aside element, rather I want to show right before your post footer.

The file you need to modify, if you follow me is source/_layouts/post.html. 
Please bear in mind that if you upgrade octopress from github automatically you may overwrite this file, so it's better for you to save your theme in the .themes directory, hacking there and then apply the theme in order to store your changes somewhere.

``` html source/_layouts/post.html
<article class="hentry" role="article">
  { % include article.html % }
  <section class="related">
    <h2>You can either find interesting</h2>
    <ul class="posts">
    { % for post in site.related_posts limit:5 % }
        <li class="related">
        <a href="{{ root_url }}{{ post.url }}">{{ post.title }}</a>
        </li>
    { % endfor % }
    </ul>
  </section>
...

```

Enjoy it!
