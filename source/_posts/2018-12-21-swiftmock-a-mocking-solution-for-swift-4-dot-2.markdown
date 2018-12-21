---
layout: post
title: "SwiftMock - A Mocking Solution for Swift 4.2"
date: 2018-12-21 11:01:20 +0000
comments: true
categories: [swift]
---

In September 2015, I tried to write a mocking framework for Swift. It was over-engineered, difficult to use and quickly abandoned; I decided instead that simple fakes would suffice.

That all changed earlier this year. I was writing much more Swift, both at home (for the _Nearly Departed_ rewrite) and in my day-job, and realised that mocks really are necessary if we want to thoroughly test how our classes interact with collaborators.

tl;dr: `SwiftMock` is [available on GitHub][github-swiftmock].

<!-- more -->

###Note about my personal test approach

I'm firmly in the "mockist TDD practitioner" camp. I exercise my system-under-test in isolation, and provide test doubles for all of its collaborators. My tests exercise the _public interface_ of the system-under-test; they care that the system-under-test did the right thing, not _why_ it did the right thing.

### Various kinds of test doubles

[Martin Fowler][martin-fowler-website] write a [great article on mocking and stubbing][martin-fowler-mocks-arent-stubs] in 2007, and explains things much better than I ever could. But briefly:

* a _test double_ is a substitute implementation of a component which your system-under-test collaborates with
* there are various kinds of test doubles, including:
  * mocks - which have explicit expectations, assert those expectations and will reject unexpected calls
  * stubs - which are primed to simply return data into your system-under-test; they feed data into your system
  * fakes - which are simple reimplementations of a collaborator interface (or Swift protocol)

Let's briefly discuss these.

#### Mock function calls

I would mock a call if the collaborator is performing a useful action on behalf of the system-under-test. In this case, my test _really does care_ that the expected function is called on the collaborator class, and should fail if that expectation fails.

Examples would be:

* telling some `User` object to `logout()`
* asking a `DataSource` object to `beginNetworkRequest()`
* asking a `Timer` object to `startTimer(delay: 5)`

Mocked function calls:

* can have any number of parameters which usually must be checked (so the test should fail if you expect `startTimer(delay: 5)` but the code actually calls `startTimer(delay: 10)`)
* often have a `Void` return type
* typically have a verb-like name

#### Stubbed function calls

A stubbed call is simply used to pass data into your system-under-test. In this case, my test _doesn't care_ if this function is called or not; it only cares that the system-under-test produces the correct output.

Examples:

* getting a `UserModel` struct from a `User` object
* getting the `isRequesting` state from an object which performs network requests
* getting the current `UserPreferences` object from a `PreferenceStore` collaborator

Stubbed functionality:

* is either a read-only property, or is a function with no parameters
* return a non-void result

This distinction between mocks and stubs is important; while you _can_ use mocks everywhere, this can lead to fragile tests. You'll eventually find yourself adding expectations that you don't care about, just to make the tests pass.

### Mixing mocked and stubbed calls in a single test double

[Martin's article][martin-fowler-mocks-arent-stubs] suggests that a test double would either be a mock, _or_ a stub, _or_ a fake - but I find this choice of mocking or stubbing depends on the _function_ being called, not to the whole object.

Take this protocol as an example of a collaborator:

    protocol User {
        var isLoggingIn: Bool { get }
        func login(username: String, password: String)
        func logout()
        func model() -> UserModel
    }

`isLoggingIn` is a read-only property, so is a great candidate for stubbing. Our system-under-test might produce some output like "Logging you in", and our test will assert that this output is generated. But it doesn't care _how_ the system-under-test produced the correct output - because that's testing the _internal_ workings of the system-under-test.

`login(username: password:)` and `logout()` functions would be mocked. Their names are verbs, suggesting that they are performing some important action on behalf of our system-under-test.

`model()` call would be stubbed. It simply returns a value, it's name is a noun, and it could be substituted for a property. My tests doesn't care if this is called or not; but if the system-under-test does call this function, my test would have some other assertions on a different observable outcome.

### How SwiftMock can help your project

_SwiftMock_ can help you assert the correctness of your code, if your tests are more in the _mockist_ style - where you substitute all of your collaborators with test doubles.

It will:

* allow you to set explicit expectations on a function call
* check the supplied parameters to the call are correct
* allow you to return values from the call back into the system-under-test
* optionally perform actions when a function is called (using a `doing` block), so function arguments can be captured
* fail-fast if unexpected calls are made

### More details

The [project README][github-swiftmock-readme] has detailed usage instructions, about how to create and use _SwiftMock_ mocks.

[github-swiftmock]: https://github.com/mflint/SwiftMock
[martin-fowler-website]: https://martinfowler.com/
[martin-fowler-mocks-arent-stubs]: https://martinfowler.com/articles/mocksArentStubs.html
[github-swiftmock-readme]: https://github.com/mflint/SwiftMock/blob/master/README.md

