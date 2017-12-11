---
layout: post
title: "Introduction to hacking iOS apps (part 5)"
date: 2017-09-03 20:41:40 +0100
comments: true
categories: [hacking,jailbreaking]
permalink: /ios-hacking-5/
---
Last time, we played with [Cycript][Cycript]. Now let's briefly explore [Frida][Frida], which is a framework for instrumenting running apps and performing code-injection.

<!-- more -->

> This is part 5 of a multi-part series:
> 
> * [Part 1][part1]: obtaining and jailbreaking a device
> * [Part 2][part2]: post-jailbreak stuff
> * [Part 3][part3]: debugging apps with lldb
> * [Part 4][part4]: altering apps at runtime with Cycript
> * Part 5: instrumentation and code-injection with Frida

### What is Frida?

[Frida][Frida] is a dynamic code instrumentation and injection toolkit. We can use it to monitor what's going on while an app is running, intercept function calls and perform arbitrary operations.

### Frida installation

There's a [complete set of installation instructions on the Frida website][FridaInstallation], but here's the tl;dr:

* open _Cydia_ on the jailbroken device
* select the _Sources_ tab
* Edit, then Add a new source: `https://build.frida.re`
* Now you should be able to search for `Frida` and install it

Finally, on your Mac, install the Frida client:

`pip install --user frida`

### Testing the installation

Connect the device to your Mac over USB, then as Frida for a list of running applications on the device:

```
Mac:~ matthew$ frida-ps -Ua
PID  Name   Identifier          
---  -----  --------------------
519  Cydia  com.saurik.Cydia    
379  Mail   com.apple.mobilemail
```

### Instrumenting function calls in a 3rd-party app

I've installed the [Amazon UK app][AmazonITunes] on the device, and we'll see what we can discover about how logins are performed. For this, we'll use [frida-trace][frida-trace].

* start the Amazon app on the device
* it looks like the "Sign In" feature happens in a modal ViewController - so we'll try looking for `viewWillAppear:` calls:
  * `frida-trace -U -f com.amazon.AmazonUK -m "-[* viewWillAppear*]"`

... which gives this output when we tap the _Sign In_ button:

```
 13506 ms  -[AWModalNavigationController viewWillAppear:0x1 ]
 13506 ms     | -[UINavigationController viewWillAppear:0x1 ]
 13507 ms     |    | -[UIViewController viewWillAppear:0x1 ]
 13517 ms  -[AIWebViewController viewWillAppear:0x1 ]
 13517 ms     | -[UIViewController viewWillAppear:0x1 ]
```

Hmm, `AIWebViewController`. Are they using an embedded `UIWebView`? Let's check:

`frida-trace -U -f com.amazon.AmazonUK -m "-[UIWebView *]"`

Whoa, loads of output there too. But a `UIWebView` which performs a login is useless without a delegate to pass information back to. We should find it.

`frida-trace -U -f com.amazon.AmazonUK -m "-[UIWebView setDelegate:]"`

... which shows this when we tap _Sign In_:

```
Instrumenting functions...                                              
-[UIWebView setDelegate:]: Loaded handler at "/Users/matthew/Dev/Temp/__handlers__/__UIWebView_setDelegate__.js"
Started tracing 1 function. Press Ctrl+C to stop.                       
           /* TID 0x503 */
  5608 ms  -[UIWebView setDelegate:0x1026737c0 ]
```

So we have a delegate being set, but no idea what kind of object that is. It probably doesn't matter, because we can easily hook into [the callback functions in the `UIWebViewDelegate` protocol][UIWebViewDelegateDocs]. We'll try hooking into `webView:shouldStartLoadWith:navigationType:`:

`frida-trace -U -f com.amazon.AmazonUK -m "-[* *shouldStartLoadWith*]"`

And sure enough, this is logged every time the _Sign In_ webpage loads content:

```
  7081 ms  -[AIWebViewController webView:0x102508130 shouldStartLoadWithRequest:0x170206510 navigationType:0x5 ]
```

The second parameter to that call is an `NSURLRequest`. I'd love to see what's in there - and we can do just that, by editing Frida's JavaScript hooks. Frida kindly generates some template hooks for us. The one we want is `__handlers__/__AIWebViewController_webView_sh_-4df00585.js`. (Yours may be named slightly differently)

If we edit this file, we can see how that previous piece of logging was performed: there are two hooks, for when the function starts (`onEnter`) and finishes (`onLeave`).


[part1]: /ios-hacking-1/
[part2]: /ios-hacking-2/
[part3]: /ios-hacking-3/
[part4]: /ios-hacking-4/
[Cycript]: http://www.cycript.org
[Frida]: https://www.frida.re
[FridaInstallation]: https://www.frida.re/docs/ios/
[AmazonITunes]: https://itunes.apple.com/gb/app/amazon/id335187483?mt=8
[frida-trace]: https://www.frida.re/docs/frida-trace/
[UIWebViewDelegateDocs]: https://developer.apple.com/documentation/uikit/uiwebviewdelegate

