---
layout: post
title: "Data-driven Screen Dispatching for iOS"
date: 2019-01-26 10:27:50 +0000
comments: true
categories: [iOS]
---

The _Coordinator Pattern_ is rather trendy at the moment. [Soroush Khanlou][soroush_coordinator_blogpost] and [Paul Hudson][paul_hudson_blog] have both written very fine blogposts about it, and I saw Paul talk about coordinators - briefly - at [iOSDevUK][iosdevuk] in 2018. There are some great benefits from using Coordinators, but I can't help feeling it doesn't go far enough.

I propose that app navigation should be driven by _data_.

<!-- more -->

### Data-driven navigation in a nutshell

My app doesn't want to show ViewControllers to the user; it wants to show _data_.

### Architectural background for my apps

* I generally use a variation of [MVVM][mvvm_wikipedia], so user-interface state is held outside the ViewController
* this means my ViewControllers are always small and free of logic

Consequently, I really don't care about ViewControllers. I don't want to create them, I don't want to present them, I don't want to write code to wire them up.

### Data-driven navigation example

In my _Nearly Departed_ rewrite, I have a `RouteModel` which defines a pair of rail stations:

    struct RouteModel {
        let fromStationCode: String
        let toStationCode: String
    }

My ViewModels will have a reference to a _dispatcher_ object (more on that later), and will present a route like this:

    let yorkToLeeds = RouteModel(fromStationCode: "YRK", toStationCode: "LDS")
    dispatcher.dispatch(yorkToLeeds)

That's it. We're showing _data_ to the user, not a ViewController.

The `RouteModel` can be dispatched because of an extension which makes it conform to a `Dispatchable` protocol - and it's that extension which decides how the data is shown to the user:

    extension RouteModel: Dispatchable {
        var dispatch: DispatchChange {
            let viewModel = DeparturesViewModel(route: self)
            return DispatchChange(presentation: .modal,
                                  options: .embedInNavigationController,
                                  viewModel: viewModel,
                                  viewControllerType: DeparturesViewController.self)
        }
    }

### Why?

There are several nice outcomes of this:

* my ViewModels can use a dispatcher to show data to the user; they don't care about _how_ it's shown
* it's easily unit-testable; I can use [SwiftMock][swiftmock_blog] to create a mock for the `Dispatching` protocol, and easily assert that the correct data is being presented
* it still has the benefits of Soroush's coordinators - where ViewControllers are completely decoupled from each other

### Source

There's an [example implementation in this Gist][example_gist].


[soroush_coordinator_blogpost]: http://khanlou.com/2015/10/coordinators-redux/
[paul_hudson_blog]: https://www.hackingwithswift.com/articles/71/how-to-use-the-coordinator-pattern-in-ios-apps
[iosdevuk]: https://www.iosdevuk.com
[mvvm_wikipedia]: https://en.wikipedia.org/wiki/Model–view–viewmodel
[swiftmock_blog]: /swiftmock-a-mocking-solution-for-swift-4-dot-2/
[example_gist]: https://gist.github.com/mflint/30d70560722ec151202cc89f824da0ee
