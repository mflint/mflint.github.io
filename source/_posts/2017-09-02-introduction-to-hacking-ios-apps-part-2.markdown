---
layout: post
title: "Introduction to hacking iOS apps (part 2)"
date: 2017-09-02 12:12:00 +0100
comments: true
categories: 
---

> This is part 2 of a multi-part series:
> 
> * [Part 1][part1]: obtaining and jailbreaking a device
> * Part 2: post-jailbreak stuff

### Recap

In the [first part of this series][part1], we acquired and jailbroke an iOS device. In part 2, we'll perform some updates and install necessary tools onto the device.

Remember that you may need to re-jailbreak your device every time it boots from cold.

### Cydia updates

When you start Cydia for the first time, you'll probably be prompted to perform an _Essential Upgrade_. I'd go for the "Complete Upgrade"; Cydia will kill itself when this is complete.

Next, we'll access the device over _ssh_.

### ssh access over WiFi

The yalu jailbreak already includes the [Dropbear][Dropbear] ssh server, so no need to install _OpenSSH_. By default, the server is only enabled over USB - so we'll need to change the config to allow this over WiFi.

* use Cydia to install "Filza File Manager" app
* open Filza, and find "/private/var/containers/Bundle/Application/yalu102/yalu102.app/dropbear.plist"
* use the (i) button to change "OPen with" to "Text Editor", then tap "dropbear.plist" to edit it
* find `127.0.0.1:22` and replace it with `22` (this binds the dropbear ssh daemon to all network interfaces, instead of the default which just uses loopback)
* reboot the device (and re-jailbreak if necessary)
* you should be able to ssh to the device
  * `ssh root@{ip-address}`
  * password `alpine`
* now's a great time to change the root password
  * `passwd`
* and also change the password for the user "mobile", which is used for running most apps
  * `passwd mobile`

### debugserver

In order to debug applications running on the device, we'll to install need the `debugserver` tool. This is dead easy to do - Xcode can install it for us.

* open Xcode and create a new project
* make sure the deployment target is low enough to run on the device
* debug the new app on the device
* Xcode will install `debugserver` for you
* check it's installed on the device:
  * `ls -l /Developer/usr/bin/debugserver`

Next time: we'll debug a running app, and make some runtime changes.


[part1]: /blog/2017/09/02/introduction-to-hacking-ios-apps-part-1/
[Dropbear]: https://matt.ucc.asn.au/dropbear/dropbear.html

