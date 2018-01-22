+++
date = "2015-07-25T17:01:17+02:00"
title = "Wercker step to minify static resources"
tags = ["CI", "Wercker"]
+++

I recently blogged a lot about [static websites]({{< relref "post/an-introduction-to-hugo-a-static-site-generator.md" >}}) and [continuous integration]({{< relref "post/continuous-integration-with-hugo-and-wercker.md" >}}). There was still one step missing in my continuous integration cycle: **minification**.

[Wercker](http://wercker.com/) is a great CI tool and it allows you to create custom build steps that can be reused in several projects. So I created a build step to minify HTML, CSS and JS files.

The build step is available at https://github.com/SamuelDebruyn/wercker-step-minify and all you have to do is include `samueldebruyn/minify` as a step in your *wercker.yml* file.
