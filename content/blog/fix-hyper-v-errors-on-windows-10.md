+++
date = "2015-10-11T22:38:46+02:00"
draft = false
title = "How to fix common Hyper-V errors on Windows 10"
tags = ["Windows 10", "Hyper-V", "Docker", "virtualization"]
+++

This short post is more of a note to myself for when I run into the same problem. I just wasted an hour on it so I guess that justifies it.

So, what happened? I wanted to check out the new [Docker Machine](https://docs.docker.com/machine/) with Hyper-V support, but the Hyper-V Manager told me it could not connect to my local computer.

When I opened Hyper-V Manager all I saw was an introduction. I then clicked *Connect to server* and selected my local computer. A confusing error message appeared telling me that an object was not found. It was the *Hyper-V Virtual Machine Management service* which seemed to be missing.

My hypervisor was running, the mentioned service was running, there were no errors in the Event Viewer... everything seemed to be working fine.

A lot of Googling led me to [a post](https://community.spiceworks.com/how_to/122307-fix-error-managing-hyper-v-server-2012-r2-from-windows-10) mentioning a command that fixed this problem:

	MOFCOMP %SYSTEMROOT%\System32\WindowsVirtualization.V2.mof
	
This command reinitializes Hyper-V management on your computer and fixes this error. It seems to be a frequent cause of headaches on Windows 10 hosts.