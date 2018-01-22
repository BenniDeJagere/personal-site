+++
date = "2015-07-15T13:21:45+02:00"
title = "Wercker step to validate Hugo themes"
tags = ["CI", "Hugo", "Wercker"]
+++

So last week, I created [a material design theme]({{< relref "blog/material-lite-theme-for-hugo.md" >}}) for [Hugo](http://gohugo.io), a static site generator which I'm [quite fond of](/tags/hugo/).

As I [discovered Wercker]({{< relref "blog/continuous-integration-with-hugo-and-wercker.md" >}}), an awesome CI tool, I went looking for a way to automatically validate themes.

There wasn't any, so I simply wrote a build step for [Wercker](http://wercker.com). The build step validates a Hugo theme using [an example site](https://github.com/spf13/HugoBasicExample). It also checks if you included some files required for a future Hugo themes site. 

The code for the build step is available at https://github.com/SamuelDebruyn/wercker-step-hugo-theme-check and an example *wercker.yml* configuration is included below.

```YAML
box: debian
build:
	steps:
	- samueldebruyn/hugo-theme-check:
		version: "0.14"
		theme: material-lite
```
