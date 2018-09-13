---
layout: post
title: "Create an highlight octopress plugin"
date: 2012-08-28 08:31
comments: true
published: true
featured: false
categories: octopress builders ruby plugin 
hn: 
rd: 
---

Suppose you're writing highlight something very important, and something
less important that it won't to you to win the [Turing award](http://en.wikipedia.org/wiki/Turing_Award). 
And after awhile, you write highlight another piece of text that you really want to highlight.

and something that people can soon forget.

How can you make it happen if you blog with [octopress](http://octopress.org/)?
You can use [my highlight plugin.](https://github.com/thesp0nge/octopress_highlight_plugin)

<!-- more -->

Writing an octopress plugin is very easy, this is the second one after a
[gravatar plugin](https://github.com/thesp0nge/octopress_gravatar_plugin) I
wrote in May to make you to use gravatar keyword in your posts.

Octopress uses [Jekyll](https://github.com/mojombo/jekyll) static website
creation framework and [liquid](http://liquidmarkup.org/) templating system for
ruby.

``` ruby Highlight plugin source code
module Jekyll
  class Highlight < Liquid::Tag
    @text = ""

    def initialize(tagname, text, tokens)
      @text = text
    end

    def render(context)
      "<span class=\"fluo\">#{@text}</span>"
    end

  end


end
Liquid::Template.register_tag('highlight', Jekyll::Highlight)
```

This is the piece of css I placed in my sass/custom/_layout.scss file.

``` css
span.fluo {
  background-color: #FEDB7C;
  padding: 3px 1px 3px 1px;
  text-shadow: 0 -1px 1px #DAC793;
}
``` 

Enjoy it!
