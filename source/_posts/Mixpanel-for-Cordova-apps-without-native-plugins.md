---
title: Mixpanel for Cordova apps without native plugins
date: 2016-04-25 21:48:09
tags:
---

Mixpanel is a tracking and analytics platform that allows you to track and analyse user behaviour in your apps.

![Mixpanel](/images/mixpanel.svg)


Mixpanel provides great libraries for [Web](https://mixpanel.com/help/reference/javascript), [iOS](https://mixpanel.com/help/reference/ios), and [Android](https://mixpanel.com/help/reference/android).
But what about cross-platform web applications that run in [Cordova](https://cordova.apache.org/)? This is the problem I was faced with when I tried to integrate Mixpanel analytics into my simple [prayer times app](/dev/ionic-speed-writing-a-prayer-times-smartphone-app-in-a-day/).

## Why not use the Mixpanel JS library?

The most obvious solution is to use the [JS API](https://mixpanel.com/help/reference/javascript) that is provided directly from Mixpanel. However, there's a problem with this approach. The JS API assumes always-on connectivity. However, as we know, mobile phones aren't always on. If you want to track user behaviour while they're on a plane or when they don't have a connection, you can't use the JS library. [Mixpanel even say this in a blog post](https://mixpanel.com/blog/2014/08/18/integrating-mixpanel-with-cordova).
 
## Wrap iOS and Android with JavaScript

To track events using Mixpanel while the user is offline, you need to implement a queuing system. The iOS and Android libraries implement this. In fact, there is a [Cordova plugin](https://github.com/samzilverberg/cordova-mixpanel-plugin) that hooks into the official Mixpanel iOS and Android libraries - so you can track mixpanel events using 
 
Although this will work, there are a few problems with this approach: 

1. Your app will depend on three libraries: the iOS, the Android, and the wrapper libraries. You'll need to manage these dependencies properly as updates come in. This will also increase the size of your app.
2. Other platforms will not be supported. For instance, Windows Phone will not be supported unless a wrapper for that comes out.
3. Performance considerations. It's good to bear in mind that calling Java and Objective C from JavaScript is not a smooth process, and it will include serializing and unpacking data you send to and from the native libraries. This may give an overhead when tracking events.

## Mind the queue: write your own Mixpanel library

Luckily, we don't have to settle for a native wrapper. Mixpanel also provide a RESTful [HTTP API](https://mixpanel.com/help/reference/http#tracking-via-http). We can implement our own Mixpanel library that allows for offline tracking.

