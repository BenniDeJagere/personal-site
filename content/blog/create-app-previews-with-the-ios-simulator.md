+++
date = "2018-10-29T14:38:56+00:00"
title = "Create App Previews with the iOS Simulator"

+++
I don't always have a ton of iOS devices lying around when I submit iOS apps to ~~iTunes~~ App Store Connect.

So I take my screenshots on the simulators. At the time of writing Apple requires screenshots made with an iPhone 8 Plus and an iPad Pro. You can optionally add screenshots from an iPhone XS Max. Apple then downscales them to all other iPhones or you can add screenshots for the other models yourself.

Apple recently introduced [App Previews](https://developer.apple.com/app-store/app-previews/). These are short videos that demonstrate how your app works. These can also be created using the simulator.

### Screen recording from the simulator

This records the screen of your simulator and saves it to the current directory. Use CTRL+C to stop recording.

Make sure your video is between 15 and 30 seconds long.

    xcrun simctl io booted recordVideo recording.mov

### Convert to the required format

For a 5.5 inch iPhone, Apple requires 30 full HD frames per second, a stereo audio stream and the H264 format.

Use homebrew to install ffmpeg:

    brew install ffmpeg

Convert your video to the correct format using ffmpeg:

    fmpeg -f lavfi -i anullsrc=channel_layout=stereo:sample_rate=44100 -i recording.mov -vf scale=1080:1920 -map 0:0 -map 1:0 -shortest -strict experimental -r 30 -y apppreview.mp4

* `-f lavfi -i anullsrc=channel_layout=stereo:sample_rate=44100`: adds silent audio (stereo)
* `-vf scale=1080:1920`: resizes video to 1080x1920
* `-r 30:` duplicates frames so that you have 30 FPS

Now upload your video using Safari and you're done.