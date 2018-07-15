---
layout: post
title: "Rewriting Nearly Departed (part 4: storage and sync)"
date: 2017-12-23 22:16:29 +0000
comments: true
categories: [swift, nearlydeparted]
permalink: /nearly-departed-rewrite-storage-and-sync/
---

This post gives an overview about how _Nearly Departed_ Routes are stored and synced between iPhone and Apple Watch.

<!-- more -->

> This is part 4 of a multi-part series:
>
> * [Part 1][part1]: intro
> * [Part 2][part2]: tech background
> * [Part 3][part3]: rewrite plans
> * Part 4: storage and sync

### Route storage

Routes are stored in `UserDefaults` on the iPhone, and shared with the Watch using _WatchConnectivity_. To keep things simple, I have a `Storing` protocol for storing preferences and two implementations - one for the iPhone and one for the Watch.

The iPhone implementation stores preferences in `UserDefaults` and also sends things to the Watch. The Watch implementation simply talks to its counterpart on the iPhone.

After some experimentation (and help from [Apple's WatchConnectivity documentation][apple watch connectivity] and [Kristina Fox's blog][kristina fox watch connectivity]), I settled on this communication strategy:

> * When the Watch app first starts, it requests the current preferences by sending a "get" command to the iPhone via _Interactive Messaging_:
> 
  {% codeblock lang:swift %}
  session.sendMessage(["cmd" : "get"], replyHandler: { (reply) in
    // handle reply from phone
  }) { (error) in
    // handle error
  }{% endcodeblock %}
>
> * If preferences change on the iPhone, it sends data to the Watch by updating the _Application Context_:
> 
  {% codeblock lang:swift %}
  // prefs is [String : Any]
  session.updateApplicationContext(prefs) {% endcodeblock %}
>
> * If preferences change on the Watch, it sends data to the iPhone using _Interactive Messaging_:
> 
  {% codeblock lang:swift %}
  // prefs is [String : Any]
  session.sendMessage(prefs, replyHandler: nil, errorHandler: { (error) in
    // handle error
  }) {% endcodeblock %}
>

This strategy seems to work well; both _Application Context_ and _Interactive Messaging_ work reliably, and quickly. (I'm testing with iOS 11.2 and WatchOS 4.2)

### Phone dependence

This means that the Watch app doesn't store any preferences locally, but instead gets preferences from the iPhone every time the Watch app starts. This is fine for now, but will be a problem when cellular Watches are more common. I'll make a change in the future to store preferences locally on the Watch, so the app can start without a nearby phone.

[apple watch connectivity]: https://developer.apple.com/library/content/documentation/General/Conceptual/WatchKitProgrammingGuide/SharingData.html
[kristina fox watch connectivity]: https://kristina.io/watchos-2-how-to-communicate-between-devices-using-watch-connectivity/

[part1]: /nearly-departed-rewrite-intro/
[part2]: /nearly-departed-rewrite-tech-background/
[part3]: /nearly-departed-rewrite-plans/
[part4]: /nearly-departed-rewrite-storage-and-sync/

