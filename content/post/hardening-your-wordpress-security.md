+++
date = "2015-08-23T22:30:51+02:00"
title = "Hardening your WordPress security"
tags = ["WordPress", "security"]
+++

## Introduction

This is the second part of a guide about [recovering a hacked WordPress]({{< relref "post/recovering-a-hacked-wordpress-and-hardening-your-security.md" >}}) and hardening your WordPress security.

In the previous part, I showed you how to get back up and running after your website has been hacked. In this part I'll demonstrate how you can avoid being hacked by making your WordPress more secure. I always try to be exhaustive, but if you have any suggestions, corrections or feel like anything is missing, I would like you to [let me know](mailto:s@muel.be?subject=Suggestions%20for%20the%20post%20about%20securing%20Wordpress).

Even if you're not hacked, I still advice you to read the first part too. It contains some information about how attackers work, what they try to achieve and how you can make sure they lose access to your website.

<small>I planned on writing this guide a lot earlier and it's been a while since I've looked at my WordPress websites (which is not a good practice...) I'll update this guide in about a month.</small>

While this guide focusses on security, some measures could impact the performance of your website negatively while others should impact your performance positively. It's up to you to consider which measures tips you should implement and which you shouldn't.

Now, let's get started, shall we?

## Consider to use a cloud proxy

Without a good CDN and caching strategy, it's impossible to survive a DDoS attack. The easiest solution is a reverse cloud proxy like [CloudFlare](http://www.cloudflare.com). CloudFlare caches your content and allows for very precise configuration. They will also keep your static files online if your server went offline. CloudFlare offers very good protection against malicious visitors and is awesome at detecting and mitigating DDoS attacks.

They offer [a WordPress plugin](https://wordpress.org/plugins/cloudflare/) that helps with the reverse proxying.

### Encryption

CloudFlare, the tool I mentioned above, offers free server-client SSL. It isn't as secure as full SSL (SSL from your server to Cloudflare's servers and from CloudFlare's servers to your visitor's browsers), but it's a lot safer than plain HTTP.

## Make sure the web servers are secure

If you're on shared hosting, you don't have much to say about this, but you can still take measures to ensure your management portal access is safe. Use very strong passwords to manage your web server, make sure everything (PHP, MySQL, Apache...) is up to date, only allow database access from your web server (usually localhost)...

Another perspective to look at this is securing access to your files. Store your backups in a safe location, make sure that the WordPress folders and files have the right permissions (this changes sometimes, take a look at the [official website](https://codex.wordpress.org/Changing_File_Permissions) to find out which permissions you should use).

## Use strong authentication and authorisation

Use strong passwords, make sure your wp-config.php is only stored on the webserver itself, consider using [MFA](https://wordpress.org/plugins/wordpress-2-step-verification/)... There are a lot of ways to make sure that it isn't too easy to get in.

## Consider using a 3rd party email provider

Most hackers don't even want to hack your website itself. Usually they'll want to use your mail server (which is usually running on the same server) and domain to sell them to spammers. A third party email service like [Mandrill](http://mandrillapp.com/) or [SendGrid](https://sendgrid.com/) could come in handy to avoid such attacks. I recommend disabling the local mail server and creating SPF and DKIM records that only emails from your email provider. It's quite easy to setup Mandrill with WordPress with [this Mandrill plugin](https://wordpress.org/plugins/wpmandrill/).

## Always use the latest versions

[PHP is a terrible programming language.](http://eev.ee/blog/2012/04/09/php-a-fractal-of-bad-design/) It's incredibly easy to write a lot of bugs and never notice them. If a plugin or theme is updated, install the update. Every possible fix for security issues is welcome.

## Uninstall plugins and themes you don't need

If you don't necessarily need a plugin or theme, uninstall it completely. Don't even leave it deactivated. Hackers can run code in deactivated plugins without any hassle.

## Schedule security scans

While [Exploit Scanner](https://wordpress.org/plugins/exploit-scanner/) is perfectly capable of scanning a WordPress installation for (possible) exploits, you can't use it to schedule automatic scans. [Sucuri Security](https://wordpress.org/plugins/sucuri-scanner/) is definitely a must-have for every WordPress website. You can schedule automatic scans for modified files, notifications for about everything that happens on your website, thorough measures to further harden your website's security...

## Log bad behaviour

There's an awesome plugin called [Bad Behavior](https://wordpress.org/plugins/bad-behavior/) which logs malicious looking behaviour and notifies you if you want to. If you suspect a hacker is trying to get in, install this plugin. You can then use to automatically block some of his attempts.

If you notice some brute force attacks on your *wp-login.php* page, consider installing [Limit Login Attempts](https://wordpress.org/plugins/limit-login-attempts/). You can fine-tune its settings so that it's not too hard on your users.

## Change the defaults

WordPress allows you to change a lot of the default settings, here are a few things you could change to improve security by obscurity:

* Store the *wp-config.php* file a level higher than the root of your WordPress installation
* [Move the *wp-content* folder](https://codex.wordpress.org/Editing_wp-config.php#Moving_wp-content_folder)
* Don't use the `wp_` table prefix
* Use a different [login url](https://wordpress.org/plugins/sf-move-login/)
* Delete the user that was created during the initial setup and create a new one
* Never use the username *admin*
