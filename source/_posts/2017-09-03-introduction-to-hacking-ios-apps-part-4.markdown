---
layout: post
title: "Introduction to hacking iOS apps (part 4)"
date: 2017-09-03 11:58:24 +0100
comments: true
categories: [hacking,jailbreaking]
permalink: /ios-hacking-4/
---
OK team, it gets slightly more exciting now: lets tweak an iOS app at runtime, using Jay Freeman's _Cycript_ tool.

<!-- more -->

> This is part 4 of a multi-part series:
> 
> * [Part 1][part1]: obtaining and jailbreaking a device
> * [Part 2][part2]: post-jailbreak stuff
> * [Part 3][part3]: debugging apps with lldb
> * Part 4: altering apps at runtime with Cycript
> * [Part 5][part5]: instrumentation and code-injection with Frida

### What is Cycript?

Cycript is a tool based around a JavaScript interpreter, which can inject itself into another running process. It provides a rich set of features to bridge between C, C++, Objective C (and Java, if that's your thing) into JavaScript. You can use it for inspecting and manipulating running processes.

### Download Cycript and copy to the device

Let's start by going to the [Cycript homepage][Cycript] and downloading the SDK. I assume it's unzipped into your Downloads folder. Then we can copy it over to the device:

`tar -czf - ~/Downloads/cycript_0 | ssh root@192.168.1.20 'tar -xzf - -C /var/root/'`

(This will take a minute to complete - be patient)

The [Cycript manual][CycriptManual] is a great detailed read - I recommend spending 30 minutes to read the whole thing. For now, let's hook it up to the Calculator app and see what it can do.

### Cycript the Calculator app

Launch the Calculator app on the device, and ssh into it. _Cycript_ can attach to processes by _pid_ or by name. I'm weird and like to use the _pid_, but here's how you'd do it both ways:

`./cycript_0/cycript -p 520`

`./cycript_0/cycript -p Calculator`

When you've finished with _Cycript_, you can exit by typing `?exit` or send the `EOF` character (^D).

We'll start with a simple query to see what's on the screen. Let's use the `choose` command to find the some ViewController classes:

```
cy# choose(UIViewController)
[#"<Calculator.CalculatorController: 0x100205170>",
#"<Calculator.DisplayViewController: 0x1002054c0>",
#"<Calculator.KeypadViewController: 0x100205a10>"]
```

Interesting! Looks like there's a main `CalculatorController` and two child ViewControllers for the the display and the keyboard. Sweet.

So what about labels?

```
cy# choose(UILabel)
[#"<Calculator.CalculatorKeypadLabel: 0x10030c540; baseClass = UILabel; frame = (160 1.5; 79 79); text = '%'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17408c7b0>>",
#"<UILabel: 0x10030da30; frame = (257.5 68.5; 47.5 99.5); text = '9'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17408bc70>>",
#"<UILabel: 0x100206eb0; frame = (18 142.5; 0 0); userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17008c120>>",
#"<Calculator.CalculatorKeypadLabel: 0x10020d080; baseClass = UILabel; frame = (1.5 1.5; 78.5 79); text = 'C'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17008cad0>>",
#"<Calculator.CalculatorKeypadLabel: 0x10020d830; baseClass = UILabel; frame = (80.5 1.5; 79 79); text = '\xef\xbf\xbc'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17008ce40>>",
#"<Calculator.CalculatorKeypadLabel: 0x10020ff90; baseClass = UILabel; frame = (1.5 81; 78.5 79); text = '7'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17008ce90>>",
#"<Calculator.CalculatorKeypadLabel: 0x1002105c0; baseClass = UILabel; frame = (239.5 1.5; 79 79); text = '\xc3\xb7'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17008cee0>>",
#"<Calculator.CalculatorKeypadLabel: 0x1002108c0; baseClass = UILabel; frame = (80.5 81; 79 79); text = '8'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17008d1b0>>",
#"<Calculator.CalculatorKeypadLabel: 0x100210b50; baseClass = UILabel; frame = (160 81; 79 79); text = '9'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17008cfd0>>",
#"<Calculator.CalculatorKeypadLabel: 0x100210de0; baseClass = UILabel; frame = (239.5 81; 79 79); text = '\xc3\x97'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17008d390>>",
#"<Calculator.CalculatorKeypadLabel: 0x100211070; baseClass = UILabel; frame = (1.5 160.5; 78.5 79); text = '4'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17008d480>>",
#"<Calculator.CalculatorKeypadLabel: 0x100211610; baseClass = UILabel; frame = (80.5 160.5; 79 79); text = '5'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17008d570>>",
#"<Calculator.CalculatorKeypadLabel: 0x1002118a0; baseClass = UILabel; frame = (160 160.5; 79 79); text = '6'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17008d660>>",
#"<Calculator.CalculatorKeypadLabel: 0x100211b30; baseClass = UILabel; frame = (239.5 160.5; 79 79); text = '\xe2\x88\x92'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17008d750>>",
#"<Calculator.CalculatorKeypadLabel: 0x100211dc0; baseClass = UILabel; frame = (1.5 240; 78.5 79); text = '1'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17008d840>>",
#"<Calculator.CalculatorKeypadLabel: 0x100212050; baseClass = UILabel; frame = (80.5 240; 79 79); text = '2'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17008d930>>",
#"<Calculator.CalculatorKeypadLabel: 0x1002122e0; baseClass = UILabel; frame = (160 240; 79 79); text = '3'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17008da20>>",
#"<Calculator.CalculatorKeypadLabel: 0x100212570; baseClass = UILabel; frame = (239.5 240; 79 79); text = '+'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17008db10>>",
#"<Calculator.CalculatorKeypadLabel: 0x100212800; baseClass = UILabel; frame = (1.5 319.5; 78.5 79); text = '0'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17008dc00>>",
#"<Calculator.CalculatorKeypadLabel: 0x100212a90; baseClass = UILabel; frame = (80.5 319.5; 79 79); userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17008dcf0>>",
#"<Calculator.CalculatorKeypadLabel: 0x100212d20; baseClass = UILabel; frame = (160 319.5; 79 79); text = '\xe2\x80\xa2'; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17008dd90>>",
#"<Calculator.CalculatorKeypadLabel: 0x100213c00; baseClass = UILabel; frame = (239.5 319.5; 79 79); text = '='; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x17008e010>>"]
```

Yup, there's a shedload of labels, as you'd expect. Can we change one?

Find the "5" label and get it's memory address. Mine is at `0x100211610`. We can assign a variable to this memory address, and easily change its label to _emoji poop_:

```
var label5=new Instance(0x100211610)
label5.text=@"\ud83d\udca9"
```

_Cycript_ is a great tool, and can do much more than this trivial change.

Next time: we'll have play with [Frida][Frida], an instrumentation and code-injection toolkit.

[part1]: /ios-hacking-1/
[part2]: /ios-hacking-2/
[part3]: /ios-hacking-3/
[part5]: /ios-hacking-5/
[Cycript]: http://www.cycript.org
[CycriptManual]: http://www.cycript.org/manual/
[Frida]: https://www.frida.re

