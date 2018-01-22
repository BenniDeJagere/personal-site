+++
date = "2015-03-23T00:00:00+00:00"
title = "How Google cripples outdated Android devices"
tags = ["Android", "Google"]
+++

## Why I downgraded my Nexus 5

About a year ago Google released [Android L Developer Preview](http://developer.android.com/preview/index.html). This preview of the upcoming Android Lollipop (5.0) release contained a lot of bugs and was definitely not _daily driver_ material.

When Google started updating Nexus devices to the [final version of Android Lollipop](http://officialandroid.blogspot.be/2014/10/android-be-together-not-same.html) last autumn, it was clear that not all bugs were gone. Users noticed several bugs of which the [memory leak on the Nexus 5](https://code.google.com/p/android/issues/detail?id=79729) was the worst. Power users had to reboot their device every 2 days and other users could go half a week without rebooting. Without a reboot, apps closed automatically a few seconds after opening and background services like Spotify shut down the moment you opened a memory intensive app.

The [Android 5.0.1 update](http://www.androidpolice.com/2014/12/15/nexus-5-receive-android-5-0-1-today-according-sprint-t-mobile/) came and went while fixing a few bugs, but the memory leak was still there. Apparently it had something to do with the fade out animation when you turned off your screen.

Then, a few weeks back, Google released Android 5.1. It contained some UI and functional improvements and the memory leak was gone, at last! But don’t start cheering too soon. [Android 5.1 contained another memory leak](http://www.androidpolice.com/2015/03/15/google-android-5-1-memory-leak-has-been-fixed-internally-no-timeline-for-release-yet/). The Android developers already fixed it, but at the time of writing, they still don’t have a timeframe for a release of this fix. The rebooting went on, every other day.

What if you have a car accident and you need to call emergency services?

> Oh no, wait, I have to reboot my phone first.

This drives me nuts. This seems a critical bug to me. A (smart)phone mustto be able to perform its most basic function all the time: calling.

I considered downgrading while I was on Android 5.0.1, but I quickly delayed this decision when I heard about the upcoming 5.1 update that would fix this memory leak. However, when I started noticing the second memory leak, I had it. Time to go back to the latest stable version: Android 4.4.4 KitKat.

## How I downgraded

The downgrade was quite simple: I unlocked my phone and flashed the 4.4.4 [factory image](https://developers.google.com/android/nexus/images). The next thing I did was rooting my phone, which later turned out to be necessary.

## And then it all started

One of the reasons why I downgraded, was that Android 4.4.4 has remarkably better battery life than Android Lollipop. So when my battery started draining at more than 20% an hour, I knew that something was wrong. Also, the notification about the system update was impossible to remove.

> Yes, Google, I know. Now let me go on with my life, I don’t want your update.

## Digging a little deeper

Android used to have a service called SystemUpdateService in the Google Services Framework (preinstalled system app) that would handle OS updates. It seems like this was replaced by Google Play Services. The [Google Play Services](https://developer.android.com/google/play-services/index.html) app is updated automatically via the Google [Play Store](https://play.google.com/store/apps/details?id=com.google.android.gms). This app contains another SystemUpdateService and disables the SystemUpdateService in the Google Services Framework mentioned earlier.

This service seems to cause a [wakelock](http://developer.android.com/reference/android/os/PowerManager.WakeLock.html), which makes sure that my Nexus 5 never goes to deep sleep. (Deep sleep is like a standby mode with very low power usage.) So there’s my battery drain. But how to stop it? Disabling this service doesn’t help at all. The notification disappears but the wakelock is still there.

Next step: Google and [XDA](http://forum.xda-developers.com/google-nexus-5/general/how-to-disable-ota-lollipop-wakelock-t2952845). I found out about the 3 receivers that trigger this service. Disabling these receivers would make sure that SystemUpdateService never runs and the wakelock is gone.

The receivers are to be found in 3 categories:

* After startup (runs after booting your device, obviously)
* Connectivity changed (runs when the status of your Wi-Fi or cellular data connections change)
* Secret code entered (runs when you enter a secret code into the dialer)

So yeah, it’s quite impossible to escape from this service. The commands I listed below correctly disable these receivers and related activities. It doesn’t disable the SystemUpdateService itself as this would cause wakelocks. Run them in a terminal on your device or on your PC connected with your phone over [ADB](http://developer.android.com/tools/help/adb.html). Did I mention you need to be _rooted_ to run it?

```shell
adb shell su -c pm disable com.google.android.gms/.update.SystemUpdateActivity
adb shell su -c pm disable com.google.android.gms/.update.SystemUpdateService$ActiveReceiver
adb shell su -c pm disable com.google.android.gms/.update.SystemUpdateService$Receiver
adb shell su -c pm disable com.google.android.gms/.update.SystemUpdateService$SecretCodeReceiver
adb shell su -c pm disable com.google.android.gsf/.update.SystemUpdateActivity
adb shell su -c pm disable com.google.android.gsf/.update.SystemUpdatePanoActivity
adb shell su -c pm disable com.google.android.gsf/.update.SystemUpdateService$Receiver
adb shell su -c pm disable com.google.android.gsf/.update.SystemUpdateService$SecretCodeReceiver
```
    
## Wow, that was complicated

For power users this probably wasn’t a lot of work, but this is a real annoyance for regular users. But regular users don’t buy Nexus devices, don’t they?

For years, we all [complained about fragmentation](http://www.google.com/trends/explore#q=android%20fragmentation&cmpt=q) and a lack of OS updates. Today it has become impossible to ignore an OS update due to the immense battery drain and the frustrating persistent notification.

## This post is all about Nexus devices, what about my device?

Most Android devices come with Google services like the Google Play Store. This means they also come with Google Play Services or the Google Services Framework or both. So, if you ever want to downgrade your Android device to a stock image, be sure to disable those pesky receivers…
