+++
date = "2013-12-22T00:00:00+00:00"
title = "Wake up your computer(s) with a simple email using a Raspberry Pi"
tags = ["Linux", "python"]
aliases = ["2013/12/wake-up-your-computers-with-a-simple-email-using-a-raspberry-pi/"]
+++

## Introduction

I have a [Spotify Premium subscription](https://www.spotify.com/premium/) but I also have a few CDs which are not available on streaming services. I don't like the whole process of syncing files, that's why I took a Spotify subscription in the first place.  Google offers a solution with [Play Music](https://play.google.com/about/music/): you can upload up to 20,000 songs to their servers for free and stream them to all of your devices. I didn't use this because the Google Play Music app was too slow for my previous phone ([Google Nexus S](http://www.android.com/devices/detail/nexus-s)). So I decided to put my CDs on a computer and use that computer as a server with [Tonido](http://www.tonido.com/tonidodesktop/). This way I could stream my music over the Internet to all of my devices. As a plus I could also put all my photos on this device so that I could show them to friends or family wherever I am.

Because this was just a standard computer this would cost a lot of electricity while I wouldn't use it 90% of the time. I would only need it when I wanted to listen to a song I own which isn't available on Spotify. That's why I came up with this idea.

I want my computer to be available when I need it and to be in standby mode when I don't need it. This way the computer boots up in a few seconds when I wake it up. I want to be able to wake it up from all over the world. I have a [Raspberry Pi](http://www.raspberrypi.org/faqs#introWhatIs) which is always on and [doesn't require much power](http://www.raspberrypi.org/faqs#power). So I created a way to send an **email to a certain address which then wakes up my computer** within the minute.


## Requirements

  * working Raspberry Pi (or similar device with Linux and python) with a constant connection to the Internet
  * the computer you want to wake up and your Raspberry Pi need to be in the same local network
  * Email account (preferably Gmail or an unused mailbox)

## Setup

### On the computer

  1. Make sure the computer goes to sleep when you don't use it. In Windows you can find this setting by searching for *power*. I have my computer set to go to sleep after 3 hours of inactivity. If you have trouble putting your computer to sleep you can analyze the problems [using the powercfg command](http://technet.microsoft.com/en-us/library/cc748940(v=ws.10).aspx).
  1. Make sure the computer wakes upon receiving a magic packet. This is a special type of TCP or UDP unicast packet that is processed by the BIOS and wakes up a computer. This also works if the computer is not in standby mode. You probably have to change [a setting in your BIOS](http://www.tomshardware.com/reviews/bios-beginners,1126-8.html) (press one of the F-buttons at boot time) and in your NIC's settings. You can view your NIC's settings in Windows using [Device Management](http://windows.microsoft.com/en-us/windows-vista/open-device-manager).
  1. [Find the MAC address](http://technet.microsoft.com/en-us/library/gg252549(v=ws.10).aspx) of the connected network adapter.

### In your [Gmail](http://www.gmail.com) account

Create a new filter for a specific email address. You can use *yourusername**+anythingyouwant**@gmail.com*

  1. Search for *to:theaddressyouchoose*
  1. Click on the little arrow left to the search button
  1. Click on the link to create a new filter in the pop-up that appears
  1. Check the following options: skip inbox, mark as read, label *new label*, never send to spam, never mark as important
  1. Enable IMAP (Settings > Forwarding & POP/IMAP > Enable IMAP)

### On your Raspberry Pi

I assume you know how to log in to your Raspberry Pi in command line, over SSH or directly. There is a lot of documentation [available](http://elinux.org/RPi_Remote_Access) on their website.

  *  Install the [wakeonlan](https://wiki.debian.org/WakeOnLan) package using the following command:
      ```sudo apt-get update && sudo apt-get -y install wakeonlan```

  *  Use nano or another text editor to create the following file:
      ```wakeonlan 01:23:45:67:89:AB```
      *01:23:45:67:89:AB* being the MAC address of the computer you want to wake up with your email. Name the file **anynameyouwant***.sh*.

  *  Create another file:

```Python
import imaplib
import os
mail = imaplib.IMAP4_SSL('imap.gmail.com')
mail.login('YOUR USERNAME@gmail.com', 'YOUR PASSWORD')
answer = mail.select('THE LABEL YOU CHOOSE')
if int(answer[1][0]) > 0:
  os.system('THE PATH TO THE FILE YOU CREATED IN STEP 2')
  mail.store("1:*",'+X-GM-LABELS', '\\Trash')
mail.close()
mail.logout()
```

  *  Replace all the words in capitals and name the file **anythingyouwant***.py*
  *  Use `crontab -e` to edit the crontab file. Scroll to the bottom and add the following line:
      ```* * * * * /usr/bin/python PATH TO THE FILE YOU CREATED IN STEP 3```
  * Save the file using CTRL+X

## Done!

Now you can send an email to the address you choose in the Gmail part above and your computer will wake up within the minute!
