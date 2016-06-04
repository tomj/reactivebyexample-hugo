---
title: "Chaining multiple network requests"
date: "2016-05-19"
description: ""
tags:
- "networking"
- "introduction"
categories:
- "networking"
- "introduction"
---

FRP promises a lot of things.  During the first talk I saw about ReactiveCocoa, I was shown that it could chain network operations.  That sounded awesome, as I really don't like asynchronous callback hell.

So here's a recipe for how to sequentially request a `/user` network resource and then, using part of that response subsequently call a `/posts` network resource.  Take a look at the whole thing and then we'll walk through the key points.

{{< gist ee1d466cfc56c288926117fd9a07080b >}}

### What did you just do above?

- Wrap up the work for a call to `/users` in a SignalProducer.  This is a sequence of events which you want to control, hence it is a "cold" signal which in RAC is achieved with a 'SignalProducer'

- Run `flatMap` on the users SignalProducer.  If you take a look at the [docs for flatMap](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/5cfcdccdcc693e7d685b4f7b7cf2c2ccf5c59c8d/ReactiveCocoa/Swift/Flatten.swift#L732) you'll see the following:

```
public func flatMap<U>(strategy: FlattenStrategy, transform: Value -> SignalProducer<U, Error>) -> Signal<U, Error>

/// Maps each event from `signal` to a new signal, then flattens the
/// resulting producers (into a signal of values), according to the
/// semantics of the given strategy.
///
/// If `signal` or any of the created producers fail, the returned signal
/// will forward that failure immediately.
```

So in this case, the `signal` referred to is the signal from your `users` request.  It will only have 1 'event' (success with an API response, or failure) so the closure we provided for `transform` is called only once.  In this closure we will take an 'event', being the result of the `users` request, and then 'map' it to a new signal, namely the `posts` request signal which is dependent on the value of the retrieved `userID`.  Lastly we provide a 'strategy' parameter to `flatMap`.  Suffice to say, if you're just chaining vanilla network requests then this can be glossed over and you can simply specify `.Latest` for this value.  In another post, I'll cover the semantics of different 'flatten' strategies.

- `start` the resulting signal which performs the `user` request in order to 'map' its event, and then performs the `posts` request using the result.

Hope this has helped!  RAC can be challenging and difficult to understand at first, but after a while you'll find that it can make programming awesome.