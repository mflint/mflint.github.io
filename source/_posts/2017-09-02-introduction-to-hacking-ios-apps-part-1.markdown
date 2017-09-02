---
layout: post
title: "Introduction to hacking iOS apps (part 1)"
date: 2017-09-02 09:30:56 +0100
comments: true
categories: [hacking,jailbreaking]
---

There are several steps to hacking iOS apps, and I aim to cover them over a series of blog posts.

1. Acquiring and jailbreaking an iOS device
2. Installing necessary tools
3. Analysing third-party apps
4. Changing functionality

In this first post, we'll acquire and jailbreak an iOS device.

<!-- more -->

> This is part 1 of a multi-part series:
> 
> * Part 1: obtaining and jailbreaking a device
> * [Part 2][part2]: post-jailbreak stuff

### Acquiring a suitable iOS device
First, some warnings:

* Don't do this on your primary iOS device - get a separate device to jailbreak
* Don't put personal stuff on this jailbroken device. That means:
  * Keep this device away from your email accounts
  * Turn off iCloud syncing, iMessage, Facetime, etc
  * Don't access your bank accounts, PayPal, etc from this device
  * Ideally, use a separate Apple account for this device

Finding a suitable device isn't easy. Latest versions of iOS (10.3.3 at the time of writing) cannot be jailbroken, and devices cannot (usually) be reverted to a previous version of iOS. So you'll need to buy a device which is already running an old version of iOS.

[the iPhone wiki][iPhoneWiki] has a handly list of jailbreakable devices and iOS versions.

There are two places where you can easily find a device for jailbreaking: eBay and high-street _cash converter_ shops.

### Buying a device on eBay
This is the cheaper option, but you need patience. Most sellers will update the device to the latest version of iOS before listing the item on eBay. You can learn a lot from the photos attached to the listing.

**Best case scenario**: there's a photo of the _About_ screen, because many buyers like to see the IMEI. On that screen you can also see the iOS version

**Not-so-good scenario**: there are no photos of the device powered-on - you'll need to message the seller to ask them which version it's running. Don't expect to get a good reply - sellers often don't know, or they might guess

**Worst-case scenario**: the phone is obviously reset (photos show the factory-reset _Hello_ screen). In this case, it's not esy for the seller to check the iOS version

### Buying a device at "cash converters"
You know _cash converters_ - they're everywhere. Shop-windows full of pre-owned phones and tablets; bins inside the shop containing used DVDs and emoji-poop cushions.

These are better than eBay for many reasons: you can handle the devices and check them for yourself, and they are less likely to be updated to the latest version of iOS. If you speak nicely to the manager, they might check all the phones for you, if they think they'll get an easy sale. Explain that you don't care about dents, scratches and carrier-locks. The only down-side is devices here will be slightly more expensive than on eBay.

I bought a 16Gb iPhone 5s from a high-street store, running iOS 10.1.1.

### Preparing for jailbreak

First, you'll need to take precautions against automatic iOS updates. If the latest version of iOS is automatically installed, your jailbreak is toast. So I mostly fill the device with empty videos:

* disable iCloud Photo Libaray (otherwise the device will try to free space by uploading these images to iCloud - but _we want_ the device to be almost full)
* open the camera app and record a 10-minute video
* open the photos app and duplicate this video multiple times, until your device has less than 700 MB free memory
* this means that the device doesn't have enough space to download iOS updates
* you'll need to keep an eye on the available storage - duplicating videos to intentionally use space, deleting videos to free up a little space
* each copy of the 10-minute video is around 150 MB on my device

### Perform jailbreak

Next go back to [the iPhone wiki][iPhoneWiki] to check which jailbreak to use for your device. For my iOS 10.1.1 iPhone 5s I'll use [yalu 102 beta 7][yalu102], installed with Jay Freeman's [Cydia Impactor][CydiaImpactor].

* download yalu102 and Cydia Impactor
* install Cydia Impactor
* attach your device
* run Cydia Impactor and make sure the device is selected in the top selectbox
* drag the yalu IPA to Cydia Impactor
* enter your Apple Developer credentials (you may need a application-specific password if you use 2FA - which you should be doing)
* yalu102 will be signed with your developer signing identity, and installed on the device
* launch yalu102 app on the device
* press **go**
* device will restart
* when it has restarted, you'll find the Cydia app installed too
* since this is a semi-untethered jailbreak tool, you'll need to run yalu102 to re-jailbreak every time the device boots

That's enough for this post. In the next blog-post, we'll install some necessary tools onto the device.



[part2]: /blog/2017/09/02/introduction-to-hacking-ios-apps-part-2/
[iPhoneWiki]: https://www.theiphonewiki.com/wiki/Jailbreak "the iPhone wiki"
[yalu102]: https://yalu.qwertyoruiop.com "yalu102 beta 7"
[CydiaImpactor]: http://www.cydiaimpactor.com "Cydia Impactor"

