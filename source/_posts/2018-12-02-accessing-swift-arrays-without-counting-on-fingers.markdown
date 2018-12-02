---
layout: post
title: "Accessing Swift Arrays Without Counting on Fingers"
date: 2018-12-02 22:59:43 +0000
comments: true
categories: [swift]
---

I have a confession to make: I've been writing code for 35 years, but still need to count on my fingers when coding an array range-check.

It goes something like this:

* I imagine I have an array containing four elements
* I hold up four fingers
* I count "zero, one, two, three", then think "so if I ask for index four, it will break, so I need to range-check..."

OMG, 35 years. I've probably done this thousands of times, and the range-check rules still won't stick in my brain.

But here's a sneaky way to avoid that range-checking code, and I'm not ashamed to say that I use this extension for real.

<!-- more -->

### An Array Extension

This is simply an Array extension which returns `nil` if the index doesn't exist. My reasoning is this: it's easier (for me) to write `if let` or `guard let` rather than counting on my fingers, and it saves me from coding stupid off-by-one errors.

    extension Array {
        func element(at index: Int) -> Element? {
            return index < self.count ? self[index] : nil
        }
    }

And I use it like this:

    if let fifthElement = myArray.element(at: 4) {
        // do something
    }

Or like this:

    guard let fifthElement = myArray.element(at: 4) else {
        // not there
        return
    }

    // do something

Result: I do the "count on fingers" thing once per Swift project, when coding this extension, instead of several times per day.

I wonder if something like this would ever be added to the standard library?

