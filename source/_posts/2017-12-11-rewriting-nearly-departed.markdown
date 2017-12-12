---
layout: post
title: "Rewriting Nearly Departed (part 1: intro)"
date: 2017-12-11 19:01:48 +0000
comments: true
categories: [swift]
permalink: /nearly-departed-rewrite-intro/
---
I'm re-writing one of my apps, [Nearly Departed][itunes], in Swift. There are four main reasons for this:

1. it's written in Objective C, not Swift - and I desperately need some more exposure to Swift
2. the code is difficult to maintain
3. there are some features I want to add which can't easily be done with the current codebase
4. fun

[Someone][someone] asked me to blog this, so here I am. This will hopefully chronicle the rewrite, until:

* I get bored of writing blogposts
* Dev stops, because I get distracted with something else, or
* It's released in the App Store

<!-- more -->

> This is part 1 of a multi-part series:
>
> * Part 1: intro
> * [Part 2][part2]: tech background

### Background

_Nearly Departed_ is an app to show live train departures in the UK. It doesn't do timetables, journey-planning or ticket purchases: just the live departures. It has an iOS app, of course, plus a today widget which can be accessed quickly from the iPhone lock screen. For a brief period, there was an Apple Watch _glance_ too (until a WatchOS update broke the synchronisation mechanism).

Here are some slightly old screenshots to show what it looks like:

{% img left /images/nearly-departed-1-app1.jpg 320 480 title alt %} 
{% img left /images/nearly-departed-1-app2.jpg 320 480 title alt %}
{% img left /images/nearly-departed-1-widget.jpg 320 480 title alt %}

I wrote _Nearly Departed_ in mid-2015, because I couldn't find a UK train departures app that I really liked. I was using ["UK Train Times" by Agant][agant-app], which is a great app, but it wasn't quite right for me:

1. the updates were infrequent and I was worried it would stop being supported
2. there was no really quick way to access live-departure information (like an iOS today widget)
3. no Apple Watch support

If none of these things matter to you, then _UK Train Times_ is well worth the purchase price.

So, like any good developer, I started to write my own.


### Feature summary

To give a little more context about the app, here are the major features:

* Show _live departure_ information for pre-defined direct routes on the UK train network
* Display this information in iOS app or today widget
* Auto-reverse routes, based on time (morning/afternoon) or user's location (if location services are enabled)

### Major releases

1.0 (May 2015)
: First release; iOS app and today widget

1.1 (June 2015)
: Apple Watch support and service alerts

1.2 (July 2015)
: Many small UI tweaks

1.3 (April 2016)
: Rail-replacement bus services; started work on supporting WatchOS 2

1.4 (April 2016)
: Auto-reverse routes based on user's location, instead of using morning/afternoon

1.5 (September 2016)
: Shows route messages (ie, why this service is delayed or cancelled); removed Watch support

1.6 (November 2017)
: Station updates

1.7 (January 2017)
: Today widget improvements

1.8 (January 2017)
: Station updates

1.9 (March 2017)
: Show all departures from your nearest station (if location services are enabled)

1.10 (August 2017)
: Station updates

1.11 (October 2017)
: Hides slow services by default

### Next

In the next post, I'll discuss the technical implementation of the app, which should start to explain why I'm rewriting it.


[someone]: https://twitter.com/dwroboheadz
[itunes]: https://itunes.apple.com/us/app/nearly-departed/id982783760?ls=1&mt=8
[agant-app]: https://itunes.apple.com/gb/app/uk-train-times/id306687757?mt=8

[part1]: /nearly-departed-rewrite-intro/
[part2]: /nearly-departed-rewrite-tech-background/

