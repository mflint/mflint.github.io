---
layout: post
title: "Enums are meant for switching"
date: 2018-11-03 09:00:16 +0000
comments: true
categories: [swift,best practices]
---
I'm rewriting _Nearly Departed_, and working on a large iOS project in my day-job. While working on both of those projects, I've started to rely on two unofficial rules for enumerations:

1. Enumeration values should always be checked with `switch`, not `if`
2. `default:` cases in `switch` statements are bad

<!-- more -->

My reason for this is that `switch` statements without `default:` cases make the intent of the code explicit; we can be sure when reading code that the developer has properly considered every case.

*As an extra bonus*, you'll now get a compile-time warning (or error, if "treat warnings as errors" is enabled) if more cases are ever added to the enumeration. This isn't as uncommon as you might think - Apple added a new case `.provisional` to the `UNAuthorizationStatus` enumeration in the iOS 12 SDK.

It's too early to have seen any real benefits from doing this (other than annoyed colleagues after I've reviewed their pull requests) but I'm _sure_ we'll be glad of it sometime in the future.

### Exceptions to these rules

In unit-test code, I wouldn't insist on `switch`ing over an enumeration. It's quite valid to do this when asserting the result of some operation:

    guard case let MyEnumeration.firstCase(myValue) = someResult else {
        XCTFail("Expected a 'firstCase' but got \(someResult) instead")
        return
    }

    XCTAssertEqual(myValue, "expected value")


### Extra bonus half-rule with no explanation

An enumeration with more than ten values is a code-smell.

