+++
date = "2015-07-08T14:50:29+02:00"
title = "An introduction to Hugo, a static site generator"
tags = ["Hugo", "jekyll"]
+++

## A good blogging platform

What makes a good blogging platform really good? Well, that depends on the blogger. Programmers would need different features than make-up bloggers. As a programmer, I need to be able to easily integrate pieces of code with syntax highlighting in my posts and I'd prefer writing them in my favourite text/code editor instead of in a WYSIWYG editor.

## Jekyll

A static site generator like [Jekyll](http://jekyllrb.com) fulfils (almost) all my needs as a programmer/blogger:

* posts are written in [Markdown](http://daringfireball.net/projects/markdown/), Textile, HTML...
* you don't need to setup a database and any simple web server will do the job since all you have to host is a simple, static website
* once setup, you don't need to think about updates etc.
* extremely developer friendly (e.g. publish a post using the command line) and open source
* works great with source code control

However, after a while, you start to notice the little things that make Jekyll a lot less simple than it looks like.

* it's written in Ruby and some of its dependencies aren't (bug free) available on ~~all platforms~~ Windows
* setting it up can be quite bothersome, again, especially on Windows

## Hugo

A few other static site generators [have sprung up](https://www.staticgen.com) since Jekyll's success.

The one that caught my attention was [Hugo](http://gohugo.io). It has all the goodness from Jekyll, but it's way easier to setup. Hugo itself is written in [Go](http://golang.org), but they build [executable binaries](https://github.com/spf13/hugo/releases) for a lot of platforms and architectures. So you don't need to install anything else. Even compiling it yourself isn't that hard.

The process of creating a new website is very similar to Jekyll. The `hugo` command literally explains itself (`hugo help`) and the [documentation](http://gohugo.io/overview/introduction/) on its website should suffice for most users.

I've successfully migrated my [Jekyll blog](https://github.com/sdebruyn/samueldebruyn.github.io/tree/5f5719a9d4519f8fbd4cdfffa2a10b3f066401ef) to [Hugo](https://github.com/sdebruyn/personal-site) and haven't regretted it ever since.

## Disadvantages

One of the major disadvantages of Hugo, or any static site generator for that matter, is integrating **search**. You need some [JavaScript magic](http://discuss.gohugo.io/t/how-are-you-implementing-site-search/986/14) or a 3rd party search provider like [Google Custom Search Engine](https://cse.google.com) (which I went for).

The other dynamic part of an ordinary blog is **the comment section**. Again, the easiest way to *fix* this is relying on a 3rd party comments plugin like [Disqus](https://disqus.com/).

Still, these two disadvantages are easily outweighed by the advantages of a static website. Your website will survive [slashdotting](https://en.wikipedia.org/wiki/Slashdot_effect) and the only required maintenance is literally making sure your web server stays online.