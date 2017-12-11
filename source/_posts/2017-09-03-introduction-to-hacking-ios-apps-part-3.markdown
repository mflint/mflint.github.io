---
layout: post
title: "Introduction to hacking iOS apps (part 3)"
date: 2017-09-03 10:56:18 +0100
comments: true
categories: [hacking,jailbreaking]
permalink: /ios-hacking-3/
---

Now we have a jailbroken phone with dev tools installed, it's time to start playing with apps. We'll start by debugging a running stock app using `lldb`.

<!-- more -->

> This is part 3 of a multi-part series:
> 
> * [Part 1][part1]: obtaining and jailbreaking a device
> * [Part 2][part2]: post-jailbreak stuff
> * Part 3: debugging apps with lldb
> * [Part 4][part4]: altering apps at runtime with Cycript
> * [Part 5][part5]: instrumentation and code-injection with Frida

### Debugging apps with `lldb`

Start the Calculator app on the device, then`ssh` into the device (as `root`). Find the process ID ("pid") of the running Calculator app.

```
JB-phone:~ root# ps ax | grep -i calculator
717   ??  Ss     0:00.31 /Applications/Calculator.app/Calculator
719 s000  R+     0:00.01 grep -i calculator
```

So the _pid_ is 717. Yours will probably be different.

Start `debugserver` on the device, and attach it to the Calculator process by using its _pid_. I've used port 1234, but it could be anything greater than 1024:

```
JB-phone:~ root# ./debugserver *:1234 --attach=717
debugserver-@(#)PROGRAM:debugserver  PROJECT:debugserver-360.0.26.1
 for arm64.
Attaching to process 717...
Listening to port 1234 for a connection from *...
```

The debugger is attached to the Calculator process, now let's attach `lldb` to it _from your Mac_:

* start `lldb` on your Mac
* `platform select remote-ios`
* `process connect connect://192.168.1.20:1234`
  * where this is the IP address of the device, and the `debugserver` port

The output looks like this:

```
Mac:~ matthew$ lldb
(lldb) platform select remote-ios
  Platform: remote-ios
 Connected: no
  SDK Path: "/Users/matthew/Library/Developer/Xcode/iOS DeviceSupport/11.0 (15A5318g)"
 SDK Roots: [ 0] "/Users/matthew/Library/Developer/Xcode/iOS DeviceSupport/10.1.1 (14B100)"
 SDK Roots: [ 1] "/Users/matthew/Library/Developer/Xcode/iOS DeviceSupport/10.2 (14C92)"
 SDK Roots: [ 2] "/Users/matthew/Library/Developer/Xcode/iOS DeviceSupport/10.2.1 (14D27)"
 SDK Roots: [ 3] "/Users/matthew/Library/Developer/Xcode/iOS DeviceSupport/10.3.2 (14F89)"
 SDK Roots: [ 4] "/Users/matthew/Library/Developer/Xcode/iOS DeviceSupport/11.0 (15A5318g)"
 SDK Roots: [ 5] "/Users/matthew/Library/Developer/Xcode/iOS DeviceSupport/11.0 (15A5327g)"
 SDK Roots: [ 6] "/Users/matthew/Library/Developer/Xcode/iOS DeviceSupport/11.0 (15A5341f)"
 SDK Roots: [ 7] "/Users/matthew/Library/Developer/Xcode/iOS DeviceSupport/11.0 (15A5362a)"
 SDK Roots: [ 8] "/Users/matthew/Library/Developer/Xcode/iOS DeviceSupport/11.0 (15A5368a)"
(lldb) process connect connect://192.168.1.20:1234
Process 717 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = signal SIGSTOP
    frame #0: 0x0000000185a4816c libsystem_kernel.dylib`mach_msg_trap + 8
libsystem_kernel.dylib`mach_msg_trap:
->  0x185a4816c <+8>: ret    

libsystem_kernel.dylib`mach_msg_overwrite_trap:
    0x185a48170 <+0>: mov    x16, #-0x20
    0x185a48174 <+4>: svc    #0x80
    0x185a48178 <+8>: ret    
(lldb)  
```

Notice that `lldb` says the process has been stopped. It really means "suspended", as if it were at a breakpoint. Tap buttons on the Calculator app - they won't work, proving that the app is paused.

We can unpause the app by issuing the `continue` command to `lldb`:

```
(lldb) continue
Process 717 resuming
(lldb) 
```

... and you'll see the app has resumed, and buttons work once more.

If you want to continue using `lldb`, then do so - there's a ton of information on [the lldb homepage][lldb]. In the next instalment, we'll switch tools and use Jay Freeman's `Cycript` to play with this app some more.

[part1]: /ios-hacking-1/
[part2]: /ios-hacking-2/
[part4]: /ios-hacking-4/
[part5]: /ios-hacking-5/
[lldb]: http://lldb.llvm.org
