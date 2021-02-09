---
date: 2016-03-12T17:24:01-08:00
description: A BaseObject.GetTypeMaker().create("sqare") peg in a type("round", (), {}) hole
title: Python Observer Metaclass
---

## Why?

Looking back and grimacing at my Master's thesis code written in the halcyon days of graduate school when I had just started learning Python, I thought it would be a rewarding process to try and re-write it using better coding practices.

I was wrong.

At first I found a few interesting ways to re-architect the code, but when that dust clear I realized I would just be spending hours re-implementing mundane details.  So I decided to scrap and let the dead horse lay.

One thing I got out of the endeavor, though, was I crafted a nifty way to implement an observer class in Python using meta-programming techniques.

If you're familiar with the [gang of four][gangoffour], you're familiar with the [observer][observer] pattern.  The basic principle is objects can subscribe to other object's events and when a particular event happens an object can notify its subscribers.

The thing about gang of four patterns is given different idioms between languages some of the patterns don't translate as gracefully into all languages, or at the least, wouldn't be implemented the same way.  This was a fun endeavor, but I wouldn't reccomend it in the real world.

Regardless, because meta-programming is interesting and fun I thought this endeavour worth sharing at least as another example of Python meta-class mechanics.

<script src="https://gist.github.com/serdmanczyk/6a22cb9868162d0d47bb.js"></script>

## What's happening?

The functions decorated by `observers` are given an attribute marking what 
they observe.

Since `Friend`'s metaclass is `Observer` the `Observer` `__call__` method intercepts the class creation before `Friend`'s `__init__` method.

The `__call__` method on `Observer` finds all functions marked as observers and stores them in a dictionary alongside their parameter specs and adds this dictionary as an attribute to the class.  It then adds a function `receive` which is the observers receive method used by communicating classes and function to pass messages.

`__call__` calls `Super` with the modified `cls`, which invokes `Friend`'s `__init__` method, continuing our normal program.

Anytime receive is called, it's passed a parameter of message type (as string), and a **splat of the message into function arguments.  The `receive` method intelligently maps the message arguments to the functions arg spec, and invokes the mapped receiving function.

In a real use case the instance generated from Friend would be passed as a subscriber to the object needing to publish state, which would notify Friend when actions happen.  My intention with this class was my parser for XBee messages in my thesis code would notify the handler classes when messages were received, and the observing handlers would route messages accordingly via this pattern.

## A Square Peg in a Round Hole

One fallacy with this method is that *only the first* instance of `Friend` will receive messages since `__call__` only builds the observing function map on creation of the class, so latter instance's won't get their functions mapped.  There are a couple of solutions to mitigate this, but they call on more code and more rabbit holes to dig down.  At this point I felt the problem further vindicated my hunch that trying to create a generalized meta-class for the observer pattern in Python was fitting a square peg in a round hole.  Nonetheless, it was a fun opportunity to mess with meta-classes and get a better idea of what uses they have.

[gangoffour]: http://www.blackwasp.co.uk/gofpatterns.aspx
[observer]: http://www.blackwasp.co.uk/Observer.aspx
