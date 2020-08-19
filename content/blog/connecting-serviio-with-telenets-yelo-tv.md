+++
date = "2013-12-09T00:00:00+00:00"
title = "Connecting Serviio with Telenet's Yelo TV"
tags = ["DLNA"]
aliases = ["2013/12/connecting-serviio-with-telenets-yelo-tv/"]
expiryDate = "2020-08-19T00:00:00+02:00"
+++

## Introduction


Recently Belgian ISP Telenet [introduced](http://snap.telenet.be/nieuw/artikel/yelo-tv-getest-en-gekeurd) [Yelo TV](http://telenet.be/nl/yelo-tv). It's the new brand for all of their efforts <del>to </del><del>innovate</del> with television. Yelo TV includes a series of apps for almost every device (except for Windows Phone) and a [website ](http://yelotv.be/)on which you can watch live television, watch recordings or schedule recordings. 

During the coming weeks Telenet is also updating their [DigiCorders](http://telenet.be/nl/digitale-tv) to a new software version called Yelo TV. Most of the changes are in design but the new firmware will include a [DLNA](http://en.wikipedia.org/wiki/Digital_Living_Network_Alliance) renderer. This means that you can use any device with a DLNA server to stream music, photos and videos to a television connected to a Telenet DigiCorder.

I have a computer with [Serviio](http://www.serviio.org/) on it. Serviio is a free DLNA server (a Pro version with more features is available) and in my opinion it's the best available. Serviio includes support for transcoding. This way Serviio will convert every media file in a format which the DLNA renderer is able to play if necessary.

DLNA has its advantages and disadvantages, one of the latter is difficulties to detect compatible media formats. That's why Serviio uses *profiles* which define the compatible media formats for each type of DLNA renderer.

There wasn't one for Telenet Yelo TV yet so I wrote one. This profile will convert every incompatible media file so that you can stream it to your DigiCorder. You'll need a fast computer to transcode files. This profile includes support for subtitles if you enabled it in Serviio's Console.


### Update: Plex

I recently switched to Plex. A profile for Plex is available here:

{{< gist sdebruyn 5f6a7b3669706153f868 >}}

## The profile:

{{< gist sdebruyn 7944739 >}}

## Installation

This profile is included in Serviio 1.4 or newer. You don't need to change anything.

## NB: Further optimizations


Tips to get a **higher video quality** (only works for videos that have to be transcoded because of incompatibility or that include subtitles):

  * For technical users only: you could force Serviio to always transcode videos, look at [*forceVTranscoding*](http://www.serviio.org/index.php?option=com_content&view=article&id=24) and [*alwaysEnableTranscoding*](http://www.serviio.org/index.php?option=com_content&view=article&id=16).
