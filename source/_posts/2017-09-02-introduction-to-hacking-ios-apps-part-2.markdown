---
layout: post
title: "Introduction to hacking iOS apps (part 2)"
date: 2017-09-02 12:12:00 +0100
comments: true
categories: [hacking,jailbreaking]
permalink: /ios-hacking-2/
---

In this second part of the "hacking iOS apps" series, we'll perform some post-jailbreak steps - ready for remote-debugging apps on a jailbroken device.

<!-- more -->

> This is part 2 of a multi-part series:
> 
> * [Part 1][part1]: obtaining and jailbreaking a device
> * Part 2: post-jailbreak stuff
> * [Part 3][part3]: debugging apps with lldb
> * [Part 4][part4]: altering apps at runtime with Cycript
> * [Part 5][part5]: instrumentation and code-injection with Frida

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

In order to debug applications running on the device, we'll to install need the `debugserver` tool. You'll need Xcode intalled on your Mac to do this.

First, take a look inside `/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport/` - you'll see folders for different versions of iOS. Inside each folder is a `dmg` file (disk archive). Open the correct one for your version of iOS:

`hdiutil attach /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport/10.1/DeveloperDiskImage.dmg`

Then copy the `debugserver` binary somewhere:

`cp /Volumes/DeveloperDiskImage/usr/bin/debugserver ./`

Create a file called `entitlements.plist`:

`cat > entitlements.plist`

... and paste in this content:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/ PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.springboard.debugapplications</key> <true/>
    <key>run-unsigned-code</key> <true/>
    <key>get-task-allow</key> <true/>
    <key>task_for_pid-allow</key> <true/>
</dict> 
</plist>
```

Then close the file by pressing ^D (Control-D). Now re-sign `debugserver` with this new entitlements file:

`codesign -s - --entitlements entitlements.plist -f debugserver`

Finally, copy this re-signed `debugserver` binary to the device:

`tar -czf - ./debugserver | ssh root@192.168.1.20 'tar -xzf - -C /var/root/'`

Next time: we'll debug a running app, and make some runtime changes.


[part1]: /ios-hacking-1/
[part3]: /ios-hacking-3/
[part4]: /ios-hacking-4/
[part5]: /ios-hacking-5/
[Dropbear]: https://matt.ucc.asn.au/dropbear/dropbear.html

