+++
date = "2015-07-06T14:14:32+02:00"
title = "Recovering a hacked WordPress"
aliases = ["2015/recovering-from-a-hacked-wordpress/"]
tags = ["WordPress", "security"]
+++

## Introduction

A few weeks ago, I had to [repair a hacked WordPress]({{< relref "post/about-that-time-I-dealt-with-a-bunch-of-hackers-on-wordpress.md" >}}) installation.

A lot of the information I needed to successfully clean the WordPress completely of infected files was not centralized in a single guide. So I decided to write one and you are now reading it.

So, your website is hacked, what now? Let's go through the process of making sure that every infection is removed and it never happens again.

In this first part you'll recover from a hack. In a second part you'll learn how to harden the security. I'll publish [the second part]({{< relref "post/hardening-your-wordpress-security.md" >}}) a few weeks after the first part.

This post is meant to be exhaustive and clear. If you have any suggestions or feel like something is missing or unclear, [*please* let me know](mailto:s@muel.be?subject=Suggestions%20for%20the%20post%20about%20recovering%20a%20Wordpress%20hack).

The WordPress version I used to write this post is 4.2.2.

## Disabling all plugins / going into maintenance mode

The first thing you should do is avoiding further infections. **Disable EVERY plugin**. We'll reactivate them later on.

I'd also put the website in maintenance mode. This requires, ironically, a plugin called [WP Maintenance Mode](https://wordpress.org/plugins/wp-maintenance-mode/). Install it, write a quick message for your visitors and enable it.

## Take a backup

Backup your complete WordPress folder and database. You can download all the files using FTP or SSH and export the database to SQL statements using something like [phpMyAdmin](http://www.phpmyadmin.net/home_page/index.php) or the customer panel of your host (CPanel, DirectAdmin...). Don't store the backup on your webserver as the hackers could also infect this backup.

## Restore the original WordPress files

Download [a fresh copy of the version of WordPress](https://wordpress.org/download/release-archive/) you were using (which should always be the latest one). Unpack it and replace the directories *wp-admin* and *wp-includes* completely with the copy you just downloaded. Do the same with every file in the root folder and remove every file in your root that is not in the downloaded copy except for *wp-config.php*. The **only folder left untouched should be _wp-content_**. WordPress also doesn't include the *wp-config.php* file so that should also be preserved (we'll inspect this file later on in this guide).

## Clearing the cache

Delete the folders *wp-content/cache* and *wp-content/plugins/widgets*. They will be automatically recreated.

## Regenerate salts and hashes

The *wp-config.php* file contains a few hashes and salts to make sure that your cookies and sessions are unique and secure. As these could have been compromised, it's **absolutely necessary** to regenerate them. Open your *wp-config.php* file in a text editor and overwrite your new hashes with new ones. WordPress offers [an online generator](https://api.wordpress.org/secret-key/1.1/salt/) so you can easily copy and paste them.

## Upgrade to the latest version

If you're not on the latest version of WordPress, then update ASAP. I'd recommend using [these extended instructions](https://codex.wordpress.org/Upgrading_WordPress_-_Extended_Instructions) instead of the built-in updater, but the built-in updater should be fine for most infected websites.

**Upgrade all your themes and plugins** while you're at it.

## Restore the original themes and plugins

In this step we're going to restore the original theme and plugin files like we did with the original WordPress files. Download the [themes](https://wordpress.org/themes/) and [plugins](https://wordpress.org/plugins/) from wordpress.org or from the website of the publisher of the plugin or theme (for premium/payed versions). All you need to do is unpack the files into their respective directories in *wp-content/themes* and *wp-content/plugins*. Make sure there are no other files than the ones you just replaced in both of these folders.

NB: some plugins (e.g. plugins containing [drop-ins](http://wpengineer.com/2500/wordpress-dropins/)) like [WP Super Cache](https://wordpress.org/plugins/wp-super-cache/) also put PHP files in your *wp-content* directory.

## Search and delete infections in your files

The only infected files left could be stored in the subfolders of *wp-content* that we haven't looked into yet.

### languages

If your WordPress is in English, you can safely delete the contents of the *languages* folder, otherwise you should download the translation in your language and put in the *wp-content/languages* while deleting everything else in it.

### upgrade

The folder *upgrade* in *wp-content* is used by WordPress' built-in updater and should always be empty except for when you're updating of course.

### uploads and other folders

Next up: search every other folder in *wp-content* for PHP and server configuration files and delete files if you think they shouldn't be there. I suggest doing a case insensitive search for _*.php_ and _.htaccess_ and carefully inspecting the results.

```shell
find /path/to/wordpress/wp-content/uploads/ -iname "*.php" -or -iname ".htaccess"
```
	
You probably shouldn't have any PHP or .htaccess files in *wp-content/uploads/someyear/*.

Finally search all the remaining files for typical code injections. PHP injections usually contain the functions `eval` and `base64_decode`.

```shell
egrep 'eval(|decode(' -r -H /path/to/wordpress/wp-content/
```

## Search and delete infections in your database

While most infections are usually to be found in the files, hackers could also insert executable and obfuscated code into your database. Most often these are XSS injections and WordPress should protect you against them. Still, it wouldn't hurt to scan your posts and comments for injected code.

The easiest way is to connect to your database right away using something like the previously mentioned *phpMyAdmin* tool. Login, open the database and go to the SQL page so you can run some SQL commands. The select statements below will show you the posts and comments containing common injections. Use a `DELETE` query (be careful) or the *phpMyAdmin* GUI to completely delete the found posts and comments. Don't forget to change `wp_` to the prefix you use.

```SQL
SELECT * FROM wp_posts WHERE post_content LIKE '%<iframe%'
UNION
SELECT * FROM wp_posts WHERE post_content LIKE '%<noscript%'
UNION
SELECT * FROM wp_posts WHERE post_content LIKE '%display:%'
UNION
SELECT * FROM wp_posts WHERE post_content LIKE '%<?%'
UNION
SELECT * FROM wp_posts WHERE post_content LIKE '%<?php%'
```
```SQL
SELECT * FROM wp_comments WHERE comment_content LIKE '%<iframe%'
UNION
SELECT * FROM wp_comments WHERE comment_content LIKE '%<noscript%'
UNION
SELECT * FROM wp_comments WHERE comment_content LIKE '%display:%'
UNION
SELECT * FROM wp_comments WHERE comment_content LIKE '%<?%'
UNION
SELECT * FROM wp_comments WHERE comment_content LIKE '%<?php%'
```

## Using *Exploit Scanner* and *Sucuri Security* to detect further infections

Most code injections should be gone by now, but there are a few WordPress plugins available to detect everything that's left.

[Exploit Scanner](https://wordpress.org/plugins/exploit-scanner/) scans your files for possible hidden infections. Its options allow you to also scan for iframes etc., but they cause a lot of false negatives.

[Sucuri Security](https://wordpress.org/plugins/sucuri-scanner/) is a plugin I'd really recommend for any WordPress installation, whether you were hacked or not. The plugin allows you to automatically scan for changed files, notify you about any administrative action, easily reinstall every plugin and theme after a hack...

Take advantage of all possibilities of the free versions of both of them. They will make sure that your WordPress is clean and stays clean.

## Check users and passwords

One of the last steps is to check if the hacker didn't create any new users. While you're at it, change the passwords of every user with more rights than a regular subscriber.

## Restoring your website and keeping it safe

Your WordPress should now be clean of infections and you can start restoring your website. Turn maintenance mode off and deactivate the *WP Maintenance Mode* plugin. Now activate all the other plugins you used one by one. Make sure to **delete any plugin you don't really need** and check if they are still being maintained/updated. I'd also delete every inactive theme. This part of the guide is crucial since it was probably one of the plugins that was used to accomplish the hack.

## Harden your WordPress Security

A few weeks after this post, I'll publish [a follow-up guide]({{< relref "post/hardening-your-wordpress-security.md" >}}) on improving the security on your WordPress website.

## Sources

* http://wordpress.stackexchange.com/questions/19696/verifying-that-i-have-fully-removed-a-wordpress-hack
* https://wordpress.org/plugins/sucuri-scanner/
* https://wordpress.org/plugins/exploit-scanner/
* https://codex.wordpress.org/FAQ_My_site_was_hacked
* http://smackdown.blogsblogsblogs.com/2008/06/24/how-to-completely-clean-your-hacked-wordpress-installation/
* http://z9.io/2008/06/08/did-your-wordpress-site-get-hacked/
