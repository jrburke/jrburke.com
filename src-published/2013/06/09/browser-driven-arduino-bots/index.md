title: Browser-driven Arduino bots
tags: [mozilla, bots]
comments: https://github.com/jrburke/jrburke.com/issues/15
~
Want to help do something awesome? Let's connect some pipes so we can drive an [Arduino](http://www.arduino.cc/) using a web browser on a mobile device!

## Problem

I attended the [node bots hack day at JSConf 2013](http://2013.jsconf.us/schedule), and it was really inspiring. We were driving Arduinos via [Node](http://nodejs.org/) and [Johnny-Five](https://github.com/rwldrn/johnny-five/).

Right now, the Node program runs on a laptop and communicates with the Arduino over a USB cable connection. There was talk of setting up connections over bluetooth or WiFi instead, but there are a couple of hurdles for it, and I want to take a slightly different path that works with a USB cable connection:

I want to control an Arduino bot by using a [Firefox OS](https://developer.mozilla.org/en/docs/Mozilla/Firefox_OS) device connected directly to the bot via USB. Why a Firefox OS device?

* it is small and relatively inexpensive.
* it has a bunch of sensors in it already, all accessible via JavaScript in web apps.
* it has a touch screen so it allows for some higher level user interactions.
* USB host mode should be available to some of the devices that run Firefox OS.

By using a direct USB connection between the device and the Arduino, it would avoid some interference issues with trying to go over a wireless connection. For example, the node copter event was running into congested/conflicting WiFi due to the amount of copters in the room.

## Pathways

At the hack day, [Tim Caswell](https://github.com/creationix) got a Google Chrome [proof of concept](https://github.com/creationix/firmata-chrome) working using the [chrome.serial API](http://developer.chrome.com/apps/serial.html) available to Chrome apps.

Firefox has the start of a ["WebUSB" patch in bug 674718](https://bugzilla.mozilla.org/show_bug.cgi?id=674718), but the patch is old and there was discussion of difficulty getting it to work for Firefox on Windows.

I tried a [naive approach to updating the patch](https://bugzilla.mozilla.org/show_bug.cgi?id=674718#c44), but ran into trouble because some of the code depended on nsDOMEventTargetWrapperCache which is no longer in the codebase. I do not have enough Gecko code knowledge to know the new mechanism to use. Also, there was a later comment indicating it was unclear how to access USB via the lower level Firefox OS operating system, [Gonk](https://developer.mozilla.org/en-US/docs/Mozilla/Firefox_OS/Platform/Gonk).

So a possible approach to getting to the end goal:

* Get a real patch that would work for Firefox OS in [bug 674718](https://bugzilla.mozilla.org/show_bug.cgi?id=674718). If it only worked for Firefox OS and did not solve any Windows issues, I would still be happy with that patch. Perhaps the patch could land only enabled for Firefox OS and Android.
* Create a JS library that uses the WebUSB patch to do effectively what [node-serialport](https://github.com/voodootikigod/node-serialport) does for Node, and what is needed for the firmata library.
* Adapt [Johnny-Five](https://github.com/rwldrn/johnny-five/) to also run in this new environment.

## How you can help

I am very inexperienced with USB and any plumbing needed for it to work, so I am poorly suited to making progress on [bug 674718](https://bugzilla.mozilla.org/show_bug.cgi?id=674718). I believe I confirmed USB host mode is available in the Firefox OS developer phone I am using, so at least on some of the phones, the hardware capability is there.

### Know USB and Android or Gonk?

If so, then you may be able to sort out the lower levels of the USB patch code in [bug 674718](https://bugzilla.mozilla.org/show_bug.cgi?id=674718).

Firefox OS runs on top of an operating system called [Gonk](https://developer.mozilla.org/en-US/docs/Mozilla/Firefox_OS/Platform/Gonk). Gonk relies on some parts from Android, and Firefox also runs on Android, so if you see a way to get the patch to run in [Firefox for Android](https://play.google.com/store/apps/details?id=org.mozilla.firefox&hl=en), that may be useful too.

### Know Gecko code?

If so, then you may be able to freshen up the higher levels of the patch in [bug 674718](https://bugzilla.mozilla.org/show_bug.cgi?id=674718) that exposes the WebUSB capability to web apps. If you do not have time to do the patch yourself, perhaps describe the higher level steps to freshen up the patch. One issue: [how to adapt the code the depended on nsDOMEventTargetWrapperCache](https://bugzilla.mozilla.org/show_bug.cgi?id=674718#c44).

### Know a better way?

I think this is a way to get to a browser-driven Arduinos, but maybe you know of a different way? I could also be missing an important step. My main objective:

* Develop apps using JS/HTML/CSS that can be loaded on a mobile device.
* Control an Arduino with those apps, and allow user interfaces that are touch friendly.

I am targeting Firefox OS because I work at Mozilla and believe in the Firefox OS mission. Firefox OS is going to be a great, low cost option for a mobile device. It is also a web-first approach. Driving Arduinos with Firefox OS devices likely means it will work for Firefox on Android, so it opens up the Android platform too.

Using Google Chrome on an Android device might also be a route to take, but I am not sure yet if Chrome apps work on Android (chrome.serial is only available to chrome apps). In my ideal world, we get it working on Firefox (on Firefox OS and Android) and Chrome (on Android) and any JS libraries above the raw USB/serial API should try to target both. You know, the typical web with support for multiple vendors.

It would be fine to hear about approaches use a tiny web server hooked up to the Arduino that receives connections over Bluetooth or WiFi, but I consider that a secondary goal as it involves more moving parts. Definitely interesting, but I really want to focus on hooking up the mobile device directly to the Arduino. The possibility of using the mobile device sensors with the Arduino, and using HTML/JS/CSS to drive the logic is something I really want to see in action.

So, leave a comment via the link below, or if you have practical steps for [bug 674718](https://bugzilla.mozilla.org/show_bug.cgi?id=674718), comment in that bug.


