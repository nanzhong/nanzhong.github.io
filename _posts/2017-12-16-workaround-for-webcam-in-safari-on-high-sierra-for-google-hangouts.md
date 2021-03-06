---
layout: post
title: Workaround for webcam in Safari on High Sierra for Google Hangouts
date: 2017-12-16 10:05:54 -0500
tags: [macos]

twitter:
  card: summary_large_image
  image: /assets/2017-12-16-workaround-for-webcam-in-safari-on-high-sierra-for-google-hangouts/hangouts-workaround-twitter.gif
---
![Hangouts workaround for Safari on High Sierra](/assets/2017-12-16-workaround-for-webcam-in-safari-on-high-sierra-for-google-hangouts/hangouts-workaround-twitter.gif)

I switched over to using Safari about a year ago because I was fed up with Chrome draining my battery, and I also really wanted to make use of the integration with iOS that lets me sync up browsing history and hand off what I was reading on my phone. But ever since I upgraded to High Sierra, the integrated webcam on my MacBook Pro stopped working with hangouts, and since I rely so heavily on Hangouts for work, it meant I had to start up Chrome just to use Hangouts. The way it fails also feels super trolly; the green light indicating that the webcam ss being used lights up, but there's just no video output... This was annoying me to no end.

Yesterday, while evaluating video quality of a bunch of different webcams, I stumbled upon what seems to be a workaround that lets me use the built in FaceTime camera in Safari for hangouts.

The **tl;dr** is that the Google Talk plugins don't seem to be initializing the camera correctly, but if you start up some other program that does initialize the camera correctly, it will start working in Hangouts.

<!--more-->

I noticed this problem immediately after upgrading and since part of the upgrade was the jump to Safari 11, I assumed that some configuration or some of my preferences must have broken during the upgrade.

The first thing that I tried was to re-install the Google Talk plugins that are needed for Hangouts on Safari and of course the trusty turning it off/on technique; this had no effect. Some quick Googling revealed that this was a common issue with High Sierra Safari users, however, the Google product forums don't suggest any fixes or workarounds that actually work.

Next, I figured if this was an upgrade issue that messed up my configuration, why don't I just reset all the settings and try that. Unfortunately, the old "reset settings" feature that I recall Safari used to have seems to have disappeared, and the only thing that you can do automatically is clear history, cookies, caches, etc. I couldn't find anything about configuration or preferences; I ended up trying to do this manually by looking for all "safari" related things in `~/Library/Application Support` and getting rid of them. This also had no effect (quite possibly because I was doing this blindly and don't actually know if I deleted the relevant config/caches or not).

Next I decided to do some digging through the system logs to see if anything interesting related to the webcam was there that would help me understand what was going on. Thankfully, I did find something. Hope restored.

```
default	12:00:58.949578 -0500	GoogleTalkPlugin	/BuildRoot/Library/Caches/com.apple.xbs/Sources/AppleFSCompression/AppleFSCompression-96.30.2/Common/ChunkCompression.cpp:50: Error: unsupported compressor 8
default	12:00:58.949641 -0500	GoogleTalkPlugin	/BuildRoot/Library/Caches/com.apple.xbs/Sources/AppleFSCompression/AppleFSCompression-96.30.2/Libraries/CompressData/CompressData.c:353: Error: Unknown compression scheme encountered for file '/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/Exceptions.plist'
default	12:00:58.952054 -0500	GoogleTalkPlugin	/BuildRoot/Library/Caches/com.apple.xbs/Sources/AppleFSCompression/AppleFSCompression-96.30.2/Common/ChunkCompression.cpp:50: Error: unsupported compressor 8
default	12:00:58.952121 -0500	GoogleTalkPlugin	/BuildRoot/Library/Caches/com.apple.xbs/Sources/AppleFSCompression/AppleFSCompression-96.30.2/Libraries/CompressData/CompressData.c:353: Error: Unknown compression scheme encountered for file '/System/Library/CoreServices/CoreTypes.bundle/Contents/Library/AppExceptions.bundle/Exceptions.plist'
default	12:01:07.256680 -0500	GoogleTalkPlugin	*** QTCaptureSession warning: Session received the following error while decompressing video: Error Domain=NSOSStatusErrorDomain Code=-8961 "noCodecErr". Make sure that the formats of all video outputs are properly configured.
default	12:01:07.256964 -0500	GoogleTalkPlugin	*** QTCaptureSession warning: Session received the following error while decompressing video: Error Domain=NSOSStatusErrorDomain Code=-8961 "noCodecErr". Make sure that the formats of all video outputs are properly configured.
default	12:01:07.268237 -0500	GoogleTalkPlugin	*** QTCaptureSession warning: Session received the following error while decompressing video: Error Domain=NSOSStatusErrorDomain Code=-8961 "noCodecErr". Make sure that the formats of all video outputs are properly configured.
default	12:01:07.290532 -0500	GoogleTalkPlugin	*** QTCaptureSession warning: Session received the following error while decompressing video: Error Domain=NSOSStatusErrorDomain Code=-8961 "noCodecErr". Make sure that the formats of all video outputs are properly configured.
default	12:01:07.323352 -0500	GoogleTalkPlugin	*** QTCaptureSession warning: Session received the following error while decompressing video: Error Domain=NSOSStatusErrorDomain Code=-8961 "noCodecErr". Make sure that the formats of all video outputs are properly configured.
default	12:01:07.365681 -0500	GoogleTalkPlugin	*** QTCaptureSession warning: Session received the following error while decompressing video: Error Domain=NSOSStatusErrorDomain Code=-8961 "noCodecErr". Make sure that the formats of all video outputs are properly configured.
default	12:01:07.398582 -0500	GoogleTalkPlugin	*** QTCaptureSession warning: Session received the following error while decompressing video: Error Domain=NSOSStatusErrorDomain Code=-8961 "noCodecErr". Make sure that the formats of all video outputs are properly configured.
default	12:01:07.430625 -0500	GoogleTalkPlugin	*** QTCaptureSession warning: Session received the following error while decompressing video: Error Domain=NSOSStatusErrorDomain Code=-8961 "noCodecErr". Make sure that the formats of all video outputs are properly configured.
```

(I've removed the non-relevant log lines)

So clearly two things seem to be wrong.

The first seems to be some sort of (un)compression error which seems to occur when the new APFS filesystem is not properly supported (aha, seems related since this issue started after my High Sierra upgrade which included the filesystem upgrade).
```
/BuildRoot/Library/Caches/com.apple.xbs/Sources/AppleFSCompression/AppleFSCompression-96.30.2/Common/ChunkCompression.cpp:50: Error: unsupported compressor 8
/BuildRoot/Library/Caches/com.apple.xbs/Sources/AppleFSCompression/AppleFSCompression-96.30.2/Libraries/CompressData/CompressData.c:353: Error: Unknown compression scheme encountered for file '/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/Exceptions.plist'
```


The second seems to be directly related to the webcam.
```
*** QTCaptureSession warning: Session received the following error while decompressing video: Error Domain=NSOSStatusErrorDomain Code=-8961 "noCodecErr". Make sure that the formats of all video outputs are properly configured.
```
So it seems to suggest that I'm missing a codec for the webcam's video stream? That's weird though because the webcam works fine for Hangouts in Chrome, and other apps (FaceTime, Photo Booth, etc.) seem to be able to use it fine... Unfortunately I knew nothing about `QTCaptureSession`, and I also couldn't find anything relevant about it online. Seems like I hit a dead end.

Or so I thought for a few months... Last night I happened to be recording some short videos captured from different webcams to compare their video quality. That's when I realized I had never tried to use any webcam other than the one in my MacBook Pro. Out of curiosity I decided to try a Logitech webcam in Safari for a Hangout and lo and behold it worked fine.

I don't know anything about how webcam video footage is encoded or how the OS handles it, but there must be something specific about the integrated FaceTime webcam that's not working. Hoping to find out more about why the FaceTime camera work in other applications, I decided to look through the logs of other applications that were able to successfully use the camera. That yielded nothing, but it turns out I forgot to close the Hangout window I was playing around with before, and through a stroke of luck the video input source that I had left configured in that Hangout was the broken FaceTime camera; when I closed Photo Booth (the app whose logs I was digging through), I noticed that the video from my FaceTime webcam was showing up in the Hangout! How... Whaa?

Long story short, I spent the next little while trying all sorts of weird orderings of starting a Hangout + starting Photo Booth, and discovered that if you start Photo Booth **after** starting the Hangout and **after** the green light is on, the FaceTime webcam will start working in the Hangout. Also, it seems to be specific about how Photo Booth initializes the camera; doing the same thing but with the FaceTime app, which can also correctly use the camera, does not work.

I definitely prefer this to having to open Chrome specifically for Hangouts, but it's still quite a hassle. Please fix this Google; if anyone close to the team responsible for Hangouts reads this, I'm happy to provide more information on reproducing this or anything else to help fix this issue.
