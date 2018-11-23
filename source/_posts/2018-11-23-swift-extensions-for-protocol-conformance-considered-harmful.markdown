---
layout: post
title: "Swift Extensions for Protocol Conformance Considered Harmful"
date: 2018-11-23 12:11:43 +0000
comments: true
categories: [swift, best practices]
---
I've come to the conclusion that using Swift extensions to make types conform to protocols is rather an anti-pattern. Let me explain why.

<!-- more -->

### How extensions can be useful

Swift type extensions are amazing for adding extra functionality to existing types if the following two conditions are true:

1. you don't own the type
2. you could reasonably argue that the functionality is _missing_ from that type

Example: if we wanted to url-encode a string (`Hello world` would become `Hello%20World`), we might do this by extending the `String` type:

    extension String {
        func urlEncode() -> String {
            // implementation here
        }
    }

However, if you already own the type, then I'd argue that you should simply add the new functionality to the type itself.

Also, if your new functionality is _very specific to the app you're writing_, then an extension probably isn't appropriate. Example: your app has a set of railway stations which can be identified by a string 'reference code'. You might consider adding an extension like this, to get the model for a given code:

    // don't do this
    extension String {
        func stationModel() -> StationModel {
            // implementation here
        }
    }

    let aslocktonStation = "ALK".stationModel()

No, don't do that. A Swift `String` doesn't care about your app.

### Three main problems with extensions for protocol conformance

However I do object to using extensions for protocol conformance:

1. extensions alter the _public interface_ for the extended type, which breaks encapsulation (exposing the internal workings of that type), and is ripe for abuse by future lazy devs
2. it's really easy to lose code when relying on extensions everywhere
3. type-safety can be damaged

### Public interface changes

Here's an example of a class which is interested in receiving messages from a paired WatchKit app:

    extension ViewController: WCSessionDelegate {
        func session(_ session: WCSession, didReceiveMessage message: [String : Any]) {
            self.myData = self.parse(incomingMessage: message)
            self.updateUI()
        }
    }

This will work OK, but now `WCSessionDelegate` is part of the public interface of this `ViewController` class. Our ViewController's WatchKit dependency is now there for everyone to see (and possibly abuse).

Somebody, eventually, will be tempted call these delegate functions directly:

    let session: WCSession? = nil
    myViewController.session(session!, didReceiveMessage: ["foo": "bar"])

A nicer alternative solution, which was common in my Java days, would be to make a private internal type which conforms to the delegate protocol:

    class ViewController: UIViewController {
        private class MessageHandler: NSObject, WCSessionDelegate {
            private weak var viewController: ViewController?

            init(viewController: ViewController) {
                self.viewController = viewController
            }

            func session(_ session: WCSession, didReceiveMessage message: [String : Any]) {
                viewController.accept(message)
            }
        }

        private var messageHandler: MessageHandler?

        override func viewDidLoad() {
            messageHandler = MessageHandler(viewController: self)
            watchSession.delegate = messageHandler
        }

        private func accept(_ incomingMessage: [String : Any]) {
            myData = parse(incomingMessage: message)
            updateUI()
        }
    }

With this solution, the WatchKit dependency is hidden and the delegate functions aren't exposed in the public interface for the class.

### Losing code in extensions

I'm a firm believer in having one unambiguous place to put a piece of code, and that code should be easy to find. Unfortunately, when using extensions for protocol conformance, code can get lost.

In my _Nearly Departed_ app rewrite, I have sets of `Model` objects which contain data received from an API, and some `ViewModel` protocols which format that data for presentation on-screen.

Simplified example:

    struct DepartureModel {
        var destination: String
        var departureTime: String
    }

    protocol DepartureViewModel {
        func departureDescription() -> String
    }

    extension DepartureModel: DepartureViewModel {
        func departureDescription() -> String {
            return "\(departureTime) to \(destination)"
        }
    }

So the `departureDescription()` func might return something like "18:05 to Nottingham".

This felt like a fabulous idea, which I embraced - but now I struggle to find those extensions. (Admittedly this is made more difficult because I have targets for an iOS app, a Today extension and a WatchKit extension)

It's so difficult to use that I'm starting to rewrite these parts with a separate concrete `DepartureViewModel` class, which holds a reference to the `DepartureModel`:

    struct DepartureViewModel {
        let departure: DepartureModel

        func departureDescription() -> String {
            return "\(departure.departureTime) to \(departure.destination)"
        }
    }

Yes, preferring composition over interitance is still just as relevant in 2018 - and conformance-via-extensions feels similar to "inheritance" to me.

### Type-safety implications

I found that I was writing code like this:

    guard let viewModel = model as? DepartureViewModel else {
        // ooh crikey, what now?
        preconditionFailure()
    }

    label.text = viewModel.departureDescription()

Not. Pretty. At. All.

I'd favour compile-time failures over runtime crashes every time.

