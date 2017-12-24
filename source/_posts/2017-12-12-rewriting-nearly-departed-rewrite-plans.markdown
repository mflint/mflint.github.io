---
layout: post
title: "Rewriting Nearly Departed (part 3: rewrite plans)"
date: 2017-12-12 01:26:14 +0000
comments: true
categories: [swift, nearlydeparted]
permalink: /nearly-departed-rewrite-plans/
---

So here's my plan for this rewrite of _Nearly Departed_.

<!-- more -->

> This is part 3 of a multi-part series:
>
> * [Part 1][part1]: intro
> * [Part 2][part2]: tech background
> * Part 3: rewrite plans
> * [Part 4][part4]: storage and sync

### Goals

* Rewrite in Swift
* ... in a _Swifty_ way
* With a decent set of unit tests
* Same features as the existing app:
  * departure boards (all departures from a station)
  * station alerts (problems with services in this area)
  * service alerts (problems with this particular service)
* New features:
  * Show service details (the progress of an individual service during its journey)
  * Reinstate Apple Watch support
  * Architect the code so it wouldn't be impossible to add different datasources in the future

### Non-goals

* Making money - this is primarily a learning exercise


### Branching strategy

My git branching strategy is slightly unusual because I want to actually use the app daily while it's being developed, but I also want to maintain code quality and decent test coverage.

master branch
: this is the nice well-tested branch
ui-spike branch
: this is the hacky branch, where code is tweaked to make things work. `master` is regularly merged into `ui-spike`, but not the other way around


### Using Laurine for localisation

I've used [Laurine][laurine] in [another app][tldr-pages] with great success - and wanted to use it again here.

You define your `Localizble.strings` file as normal:

{% codeblock lang:swift Localizable.strings %}
"PLATFORM_NUMBER" = "Platform %@";{% endcodeblock %}

and then Laurine generates a new `Localizations` swift file, which you can use like this:

{% codeblock lang:swift Localizations.swift %}
Localizations.PLATFORM_NUMBER("4"){% endcodeblock %}


### Frameworks

Nope.

It all sounds great - putting common code in a framework which is then shared between the iPhone app / widget / Watch App targets. But it never works out well.



### Data models

**We'll start by talking about _Routes_.** A route provides data for a request, and is user-defined in one of two ways:

1. Explicitly, via the iOS app. (example: all departures from York to Leeds would be an instance of `DarwinDeparturesRoute`)
2. Implicitly, from another route. (example: when viewing departures from York to Leeds, they might tap on a service to see its progress. This `DarwinServiceRoute` would be created automatically, containing the `serviceID` for the selected service)

There is also a `NoRoutesRoute`. I'm starting to develop an irrational dislike of optionals, so try to avoid them wherever I can. So if the user hasn't yet created any routes, we have a `NoRoutesRoute` there instead. (Thanks to [Kevin Rutherford][kevinrutherford] for the inspiration to go down this route, with his "write a program with no if statements" kata)

We need something to manage our collection of routes, and that's in the `RouteManager`. The user's _explicit_ routes are stored in a collection, but we also allow other routes to pushed to form a stack, so the user can drill-down from one result to another.

This might seem complex, but it allows the user to manage scenarios like this:

* User travelling from A to B, then B to C
* User creates explicit routes "A to B" and "B to C"
* When starting their journey, they select route "A to B" to see if their service is on-time
* Then they tap on the _service_ to create an implicit Service route, which is popped onto the stack. They use this to track the progress of the service they're travelling on
* But they also want to see if the "B to C" service is running OK
* So they select the "B to C" route, and tap on a service to create a second implicit Service route, which is popped onto _that_ stack
* Then they can switch between stacks, to keep track of both _services_

{% mermaid %}
graph LR
RM[RouteManager]-->E1["Explicit route 1<br/>(example: A to B)"]
RM[RouteManager]-->E2["Explicit route 2<br/>(example: B to C)"]
subgraph stack
E2-->I2["Implicit pushed route<br/>(example: 08:45 service from B to C)"]
end
subgraph stack
E1-->I1["Implicit pushed route<br/>(example: 08:00 service from A to B)"]
end
{% endmermaid %}

This collection of stacks is stored in shared `UserDefaults`, so you could push a new route using the iPhone and then view its results on the Apple Watch.


**Next, we'll need to get fetch data from an API** for each type of route using `Entity` types which can fetch a particular kind of data. Right now there are two, but there could be more in the future:

`DarwinDeparturesEntity`
: gets OpenLDBWS departure boards for services between two stations (departures from station A calling at station B)

`DarwinServiceEntity`
: gets OpenLDBWS service information (which returns the arrival/departure times for a service at each calling-point)

**Which brings us to _results_.** Each type of route has a corresponding `ResultSet` containing a collection of `Result` objects. Those `Result` objects can represent either a _service departing from a station_, or a _calling point for a single service_. And - to enable the user to drill-down - each `Result` can have provide a child `Route`, which defines the next query.

There are three other `ResultSet` objects:

`NoRoutesResultSet`
: this is used when there are no routes defined in the app
`LoadingResultSet`
: this is returned from Entities when they're fetching data for the first time
`ErrorResultSet`
: returned from Entities if an error occurred while fetching or parsing data

With these three extra `ResultSet` objects I can define the Entity's `resultSet` property as non-optional:

{% codeblock lang:swift Entity.swift %}
public protocol Entity {
    var resultSet: ResultSet { get }
}{% endcodeblock %}


In the [next post][part4], I'll describe how route definitions are stored, and how they're synchronised between iPhone and Watch.


[kevinrutherford]: https://twitter.com/kevinrutherford
[laurine]: https://github.com/JiriTrecak/Laurine/
[tldr-pages]: https://itunes.apple.com/us/app/tldt-pages/id1071725095?ls=1&mt=8

[part1]: /nearly-departed-rewrite-intro/
[part2]: /nearly-departed-rewrite-tech-background/
[part3]: /nearly-departed-rewrite-plans/
[part4]: /nearly-departed-rewrite-storage-and-sync/

