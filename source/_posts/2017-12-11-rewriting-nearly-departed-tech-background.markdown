---
layout: post
title: "Rewriting Nearly Departed (part 2: tech background)"
date: 2017-12-11 23:13:14 +0000
comments: true
categories: [swift, nearlydeparted]
permalink: /nearly-departed-rewrite-tech-background/
---

This post will give a little of the technical background to _Nearly Departed_ and explain some of the reasons for the rewrite.

<!-- more -->

> This is part 2 of a multi-part series:
>
> * [Part 1][part1]: intro
> * Part 2: tech background
> * [Part 3][part3]: rewrite plans
> * [Part 4][part4]: storage and sync

### Headline tech features

* Written in Objective C
* Kind-of [MVVM][mvvm]
* Very little unit-test coverage
* Made with a set of discrete service classes, injected using [Objection][objection]

### Why no WatchOS support any more?

The first WatchOS support was a WatchOS 1 _glance_. Communication between Watch and iPhone used shared `NSUserDefaults` and [Darwin Notifications][darwin-notifications] were used by the two processes to notify each other that something had changed. (Remember that for WatchOS1 the Watch app process ran _on the iPhone_, effectively using the Watch as a separate screen. So Darwin Notifications could be used for interprocess comms because both processes were running on the same device)

WatchOS 2 introduced _native_ third-party apps, moving the Watch app process onto the Watch itself, so the Darwin Notification code was broken.

### OpenLDBWS API

_Nearly Departed_ uses the [OpenLDBWS][open-ldbws] ("Live Departure Boards Web Service") SOAP API from [National Rail][national-rail]. Some definitions:

Darwin
: National Rail's system for tracking the movements of passenger services
OpenLDBWS
: A SOAP API for accessing real-time information from Darwin

The API gives a _tonne_ of great real-time data, including:

* Departure/arrival boards (which return the same data you would see on displays at any of our 2500 stations
* Progress information for all services which call at these stations. This includes:
  * Scheduled arrival/departure time at each calling point
  * Actual or estimated arrival/departure time at each calling point

Hats off to them, the _OpenLDBWS_ team have done a great job. I have a couple of minor complaints (such as times being sent like `08:18` without timezone or date information) - but they've managed to nicely model a proper complex set of data. Shame it's SOAP/XML though!

### Station data

While the OpenLDBWS return some station data in the responses, I still need to maintain my own set of station data which is bundled with the app. My requirements are:

* For each station, I need the following:
  * Station name (such as "Stevenage")
  * The Computer Reservation System (CRS) code, which uniquely identifies this station in the API (example: "SVG")
  * location (latitude/longitude)
* The app needs this data in _list_ format (so it can search for stations by name or CRS code)
* But also in a format which can be searched by nearest station to a given location

The data itself comes from two sources:

`station_codes.csv`
: [from National Rail][station-codes-national-rail]
`RailReferences.csv`
: [from the UK Government NaPTAN dataset][naptan]

`station_codes.csv` is simply a list of station names and their CRS codes, whereas `RailReferences.csv` also has location data in [easting/northing][easting-northing] format. Unfortunately the data doesn't completely match between those two files, so I have a Ruby script which does a few things:

* matches data from the `station_codes.csv` and `RailReferences.csv` files
* corrects inconsistencies between the two files (NaPTAN often has incorrect CSR codes)
* adds some data which is missing from one file or the other (usually because a new station has opened but the source data hasn't yet been updated)
* filters out some non-mainline stations (such as heritage rail or ferry terminals)
* converts easting/northing locations to latitude/longitude
* creates a two-dimensional binary tree based on station locations, and writes it to a 670K `.plist` file. The tree is 12 levels deep, each level being split alternatively by longitude/latitude.

The app reads this plist into memory in its tree structure for location searches, and flattens it to an array for name/CRS searches.

### Architectural limitations

* I frequently get asked to provide support for showing _ServiceDetails_ data from OpenLWBWS - ie, data for a particular service - but the app was only designed to show _DepartureBoard_ data
* The app wasn't written to use other data sources - so if I ever wanted to add London Underground data, it would be very difficult to do so
* It's just not very well written! ;-)

### Next

In the [next post][part3], I'll discuss my high-level plans for the rewrite.

[mvvm]: https://en.wikipedia.org/wiki/Model–view–viewmodel
[objection]: http://objection-framework.org
[darwin-notifications]: https://developer.apple.com/library/content/documentation/Darwin/Conceptual/MacOSXNotifcationOv/DarwinNotificationConcepts/DarwinNotificationConcepts.html
[open-ldbws]: https://lite.realtime.nationalrail.co.uk/openldbws/
[national-rail]: http://www.nationalrail.co.uk
[station-codes-national-rail]: http://www.nationalrail.co.uk/stations_destinations/48541.aspx
[naptan]: https://data.gov.uk/dataset/naptan
[easting-northing]: https://en.wikipedia.org/wiki/Easting_and_northing

[part1]: /nearly-departed-rewrite-intro/
[part2]: /nearly-departed-rewrite-tech-background/
[part3]: /nearly-departed-rewrite-plans/
[part4]: /nearly-departed-rewrite-storage-and-sync/

