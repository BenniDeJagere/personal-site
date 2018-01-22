+++
date = "2015-06-27T17:55:40+02:00"
title = "About that time I dealt with a bunch of hackers on a WordPress"
tags = ["WordPress", "security"]
+++

So yeah, that happened. I recently took over 2 WordPress installations hosted on a [Versio](http://www.versio.uk/) shared hosting account. Versio suspended the account because it kept sending thousands of (spam) emails a day.

You guessed it right, one of the WordPress installations was hacked. The previous administrator tried to delete the infected files and changed his passwords. It took them a week to regain full control of both the WordPress installations. Once they got into one of the WordPress setups, it was just a matter of navigating through the filesystem to find and infect the other WordPress installation.

The intruders were very interested in this particular website because of the domain (TLD): [asr.ac](http://asr.ac). An academic domain has a high standing, isn't very cheap and a very rare treat for spammers. They were proud of their hack and left code injections in files called *indonesia.php*.

Then I took over. I managed tens of WordPress websites before and I had a lot more time to spare than the previous IT guy. I found out that the contact address for Versio and a bunch of other services was one of the .ac addresses, which was suspended. So we didn't even receive any warnings from Versio about abuse, the hackers had free play.

Once I regained access to [DirectAdmin](http://www.directadmin.com/) and Versio's customer panel, I immediately backed up both installations, including the databases. Exporting the content and importing it into a fresh WordPress installation was a no go since the *wp-content* directory was infected. The only option seemed to desinfect both WordPress installation and secure them.

Versio sent some notices with the files the hackers used to send out spam. They were all PHP files with obfuscated code injections in them.

Great, so I knew how they did it and what I had to look for. I simply deleted everything except for the *wp-content* directory and searched the *uploads* folder for code containing *eval(* or *decode(*. I then uploaded the default WordPress files and made sure everything worked again.

The theme we used was an outdated premium theme so I replaced that with a more popular similar looking theme which was kept up to date.

Finally I setup [CloudFlare](https://www.cloudflare.com/) with secure settings and let them challenge visitors from Indonesia. I also installed the [Sucuri Security plugin](https://wordpress.org/plugins/sucuri-scanner/) and let it scan all files twice a day for infections.

So that was all, right? Nope, a few days later they were back. This time they named their files something like *diff52.php* as this wouldn't look as suspicious as *indonesia.php*. CloudFlare also showed some blocked threats from Russia and Ukraine.

[This is an example of the code I found](/addendum/2015/obfuscated-code-from-the-wordpress-hack/) and [this is the deobfuscated version of it](/addendum/2015/deobfuscated-code-from-the-wordpress-hack/).

I repeated the whole process but I kept wondering how they could get in. The only thing I didn't check were the plugins. A few days before I reinstalled them all using Sucuri Security so they didn't contain any infections. But they had to be the cause, right? I simply deactivated and removed all of them except for the ones I installed to secure the website. That did it. It's been a week and the hackers haven't been back since.

What did I learn from all of this? When you're dealing with a hacked WordPress, **always** delete every plugin and non-default theme. The team behind WordPress releases security updates in a few hours after they find a leak. [WordPress is quite secure](https://ma.ttias.be/in-defence-of-wordpress/), it's the plugins and the themes that aren't. Themes and plugins used by a lot of people will probably get more security fixes than others. The more plugins or themes you have installed, the bigger the risk.

PS: I repeated every step on the second WordPress installation since that one was infected too.

PPS: I'll post a comprehensive guide to securing/hardening WordPress in a few weeks. A good starting point is [this guide](https://codex.wordpress.org/FAQ_My_site_was_hacked).