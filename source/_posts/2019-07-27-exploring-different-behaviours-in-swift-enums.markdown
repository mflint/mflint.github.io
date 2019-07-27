---
layout: post
title: "Exploring Different Behaviours in Swift Enums"
date: 2019-07-27 19:35:48 +0100
comments: true
categories: [swift]
permalink: /different-swift-enum-behaviours/
---

I have a dilemma; the Swift `enum` is a first-class type and, if we're to obey the laws of encapsulation, an `enum` type should own its functionality. But can we do this in a nice way, when its behaviour is different for each value in the enum?

In this post, we'll explore the old Java _type-safe enumeration_ pattern, revisit my old [enums are meant for switching][EnumsSwitching] blogpost, and try to combine the lot.

<!-- more -->

### Problem statement

My open-source [tldr-pages app][TldrGithub] shows concise man-pages for command-line commands - and some of these have variations for different platforms. For example, the command `hostname` has different documentation for macOS and Linux. So I've defined a type called `Platform` to represent a single platform.

The requirements for `Platform` are simple. It must be able to:

* be instantiated by passing in a known string (`osx`, `sunos`, etc)
* handle unknown platform strings (`vms` or `os2warp`)
* give us a name which is suitable for display

### The ugly Platform.swift

At the time of writing, `Platform.swift` looks rather like this:

    class Platform {
        /// this matches the platform names in the main tl;dr project
        var name: String
        
        /// this is the name as displayed on the screen
        var displayName: String
        
        private static let platformMapping = [
            "osx": "macOS",
            "sunos": "Solaris",
            "linux": "Linux",
            "windows": "Windows",
            "common": "Common"
        ]

        private init(name: String) {
            self.name = name

            if let mapped = Platform.platformMapping[name] {
                self.displayName = mapped
            } else {
                self.displayName = name.capitalized
            }
        }
    }

And I'm creating Platform objects like this:

    let macOS = Platform(name: "osx") // "osx" value comes from the tl;dr project

There's lots of things wrong with this type. First, it's a `class`, and there's no reason for it to be a reference type. We need a different approach.

### Changing `Platform` to be an `enum`

An `enum` would be ideal, so let's try that:

    enum Platform: String {
        case osx
        case sunos
        case linux
        case windows
        case common
        case unknown(name: String)
        
        var displayName: String {
            get {
                switch self {
                case .osx:
                    return "macOS"
                case .sunos:
                    return "Solaris"
                case .linux:
                    return "Linux"
                case .windows:
                    return "Windows"
                case .common:
                    return "Common"
                case .unknown(let name):
                    return name.capitalized
                }
            }
        }
    }

I said a while back that [enums were meant for switching][EnumsSwitching], and this satisfies my rule.

This looks nice - except it **doesn't compile**. A Swift `enum` can have a raw type (`enum Platform: String`) or a case with arguments (`unknown(name: String)`) but not both. Our requirement is that this _must_ handle unknown platform names, so we can't lose the `unknown` case. The raw type must go, but we can provide similar functionality by adopting `ExpressibleByStringLiteral`:

    enum Platform: ExpressibleByStringLiteral {
        case osx
        case sunos
        case linux
        case windows
        case common
        case unknown(name: String)
        
        var displayName: String {
            get {
                switch self {
                case .osx:
                    return "macOS"
                case .sunos:
                    return "Solaris"
                case .linux:
                    return "Linux"
                case .windows:
                    return "Windows"
                case .common:
                    return "Common"
                case .unknown(let name):
                    return name
                }
            }
        }

        // initializer for ExpressibleByStringLiteral
        public init(stringLiteral name: String) {
            switch name {
            case "osx":
                self = .osx
            case "sunos":
                self = .sunos
            case "linux":
                self = .linux
            case "windows":
                self = .windows
            case "common":
                self = .common
            default:
                self = .unknown(name: name)
            }
        }
    }

Now we can use the enum like this:

    let platform: Platform = "osx"
    platform.displayName // "macOS"

But here's the problem: we now have two ugly `switch` statements in our `Platform` type - and if we add more functionality to the type, then it's likely that we'll be adding even more `switch`es.

Worse still: all the logic for each different platform is scattered throughout the whole class, and ideally all the _linux_ code would be grouped together. I don't want switches everywhere.

### Enter the _strategy_ pattern

I first came across the [Strategy Pattern][StrategyPattern] in the [Gang of Four][GangOfFour] in the late 1990's, and this pattern is perfect for the case where every function in an object consists of a near-identical `switch` statement. It looks like this:

    // this protocol defines the common behaviours for each platform
    private protocol PlatformStrategy {
        var displayName: String { get }
    }
    
    // now we define a concrete implementation for each platform
    private struct OSXPlatformSrategy: PlatformStrategy {
        let displayName = "macOS"
    }

    // and our public Platform class has a reference to one of the platform types
    public struct Platform {
        private let strategy: PlatformStrategy

        public var displayName: String {
            get {
                return strategy.displayName
            }
        }

        init() {
            // somehow decide which platform this is
            strategy = OSXPlatformStrategy()
        }
    }

This seems like a lot of work for a single `displayName` proprety - but if there were a few more properties, there are real advantages of keeping all the _macOS_ functionality inside the `OSXPlatformStrategy` type.

In the old Java world, before `enum` was added to the languade in Java 5, we used to write _typesafe enumerations_ like this:

    class Platform {
        static final Platform osx = new Platform("osx");
        static final Platform linux = new Platform("linux");

        String name;

        private Platform(String name) {
            this.name = name;
        }
    }

... and it wasn't hard to apply the strategy pattern to the Java typesafe enum. We'd make the enum type _abstract_, then define each case as an anonymous class which extends the abstract type, and provides implementations for the abstract methods:

    abstract class Platform {
        static final Platform osx = new Platform("osx") {
            String displayName() {
                return "macOS";
            }
        };
        
        static final Platform linux = new Platform("linux") {
            String displayName() {
                return "Linux";
            }
        };
        
        String name;
        abstract String displayName();
        
        private Platform(String name) {
            this.name = name;
        }
    }

This is nice; the individual behaviour for the _osx_ and _linux_ cases is encapsulated entirely in the two anonymous classes. Can we do something similar in Swift?

### Broken Swift implementation

The equivalent Swift code would look like this:

    protocol PlatformStrategy {
        var displayName: String { get }
    }

    enum Platform {
        case osx
        case linux

        var strategy: PlatformStrategy

        var displayName: String {
            get {
                return strategy.displayName
            }
        }

        init(name: String) {
            // set the strategy here
        }
    }

Unfortunately is doesn't compile, because `enums must not contain stored properties`. However Swift enums already do (kind of) have a property: `rawValue`. We'd need to lose our `unknown` case with its argument, but can we abuse `rawValue` as a place to store our strategy?

### `rawValue` as a strategy type

It's not as easy as you'd think, because the raw value for a case must be a literal value. So this won't work:

    protocol PlatformStrategy {}

    class OSXStrategy {}

    class LinuxStrategy {}

    enum Platform: PlatformStrategy {
        case osx = OSXStrategy()     // not a literal
        case linux = LinuxStrategy() // not a literal
    }

But it _is_ possible to declare a new type for our `rawValue` which adopts `ExpressibleByStringLiteral`. (Currently, this also needs to adopt `Equatable` to avoid a Signal 11 compiler crash)

    protocol PlatformStrategy {
        var displayName: String { get }
    }

    struct OSXStrategy: PlatformStrategy {
        var displayName = "macOS"
    }

    struct LinuxStrategy: PlatformStrategy {
        var displayName = "Linux"
    }

    struct OtherStrategy: PlatformStrategy {
        var displayName: String

        init(_ name: String) {
            displayName = name
        }
    }

    struct PlatformRawValue: ExpressibleByStringLiteral, Equatable {
        let strategy: PlatformStrategy

        public init(stringLiteral name: String) {
            switch name {
            case "osx":
                strategy = OSXStrategy()
            case "linux":
                strategy = LinuxStrategy()
            default:
                strategy = OtherStrategy(name)
            }
        }

        static func ==(_ lhs: PlatformRawValue, _ rhs: PlatformRawValue) -> Bool {
            return lhs.strategy.displayName == rhs.strategy.displayName
        }
    }

    enum Platform: PlatformRawValue, RawRepresentable {
        typealias RawValue = PlatformRawValue

        var displayName: String {
            get {
                return rawValue.strategy.displayName
            }
        }

        case osx = "osx"
        case linux = "linux"
    }

    Platform.osx.displayName
    Platform.linux.displayName

In this example, we have a `LinuxStrategy` type which encapsulates all the linuxy behaviour separately from the `OSXStrategy`, and the `Platform` enum uses a custom `rawValue` type! Unfortunately I can't simulate the `unknown` case with arguments. I've tried to do this by calling `Platform(rawValue: "osx")?.displayName`, but the `Platform`initializer returns `nil`.

### Conclusion

I can't find an easy way to define a Swift `enum` where each case has distinct behaviour, unless I use `switch` statements everywhere. Sad times!

Ideally, we'd be able to do something like this:

    protocol PlatformProtocol {
        func displayName() -> String
    }

    enum Platform: PlatformProtocol {
        case osx
        case linux
    }

    extension Platform where Self == Platform.osx {
        func displayName() -> String {
            return "macOS"
        }
    }

    extension Platform where Self == Platform.linux {
        func displayName() -> String {
            return "linux"
        }
    }



[EnumsSwitching]: /enums-are-meant-for-switching/
[TldrGithub]: https://github.com/mflint/ios-tldr-viewer/
[StrategyPattern]: https://en.wikipedia.org/wiki/Strategy_pattern
[GangOfFour]: https://en.wikipedia.org/wiki/Design_Patterns
