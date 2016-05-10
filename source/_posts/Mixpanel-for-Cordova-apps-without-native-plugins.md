---
title: Offline-resilient Mixpanel tracking for Ionic without a Cordova plugin
date: 2016-04-25 21:48:09
thumbnail: /images/goneoffline.png
tags:
  - android
  - app
  - ionic
  - ios
  - mixpanel
  - cordova
  - native
  - offline
categories:
  - dev
---

Mixpanel is a tracking and analytics platform that allows you to track and analyse user behaviour in your apps.

![Mixpanel is a user behaviour event tracking library for the Web, iOS, and Android](/images/mixpanel.svg)


Mixpanel provides great libraries for [Web](https://mixpanel.com/help/reference/javascript), [iOS](https://mixpanel.com/help/reference/ios), and [Android](https://mixpanel.com/help/reference/android).
But what about cross-platform web applications that run in [Cordova](https://cordova.apache.org/)? This is the problem I was faced with when I tried to integrate Mixpanel analytics into my simple [prayer times app](/dev/ionic-speed-writing-a-prayer-times-smartphone-app-in-a-day/).

## Why not use the Mixpanel JS library?

The most obvious solution is to use the [JS API](https://mixpanel.com/help/reference/javascript) that is provided directly from Mixpanel. However, there's a problem with this approach. The JS API assumes always-on connectivity. However, as we know, mobile phones aren't always online. If you want to track user behaviour while they're on a plane or when they don't have a connection, you can't use the JS library. [Mixpanel even say this in a blog post](https://mixpanel.com/blog/2014/08/18/integrating-mixpanel-with-cordova).
 
## Wrap iOS and Android with JavaScript

To track events using Mixpanel while the user is offline, you need to implement a queuing system. The iOS and Android libraries implement this. In fact, there is a [Cordova plugin](https://github.com/samzilverberg/cordova-mixpanel-plugin) that hooks into the official Mixpanel iOS and Android libraries - so you can track mixpanel events using 
 
Although this will work, there are a few problems with this approach: 

1. Your app will depend on three libraries: the iOS, the Android, and the wrapper libraries. You'll need to manage these dependencies properly as updates come in. This will also increase the size of your app.
2. Other platforms will not be supported. For instance, Windows Phone will not be supported unless a wrapper for that comes out.
3. Performance considerations. It's good to bear in mind that calling Java and Objective C from JavaScript is not a smooth process, and it will include serializing and unpacking data you send to and from the native libraries. This may give an overhead when tracking events.

## Mind the queue: write your own Mixpanel library

Luckily, we don't have to settle for a native wrapper. Mixpanel also provide a RESTful [HTTP API](https://mixpanel.com/help/reference/http#tracking-via-http). We can implement our own Mixpanel library that allows for offline tracking.

The trick to offline event tracking is to keep a queue of things that you're going to send to Mixpanel. We can use the http API from mixpanel to send the events.

```js
  var queueBuffer = [];
  function pushToQueue(val){
      val.id = queueBuffer.push(val) + (new Date().getTime());
      return val.id;
   }


  function track(event, properties){
    var nowTime = new Date().getTime();
    pushToQueue({
      event: event,
      properties: _.merge({time: nowTime}, registrationProperties, properties || {}),
      timeTracked: nowTime,
      endpoint: 'track'
    });

    if(queueBuffer.length > 4){
      push();
    } else {
      schedulePush();
    }
  }
```

### Not online? Wait in the queue!

Periodically, we attempt to send 4 items in the queue. If the send succeeded, we remove those 4 items and continue with the next items in the queue. If the send failed, we keep the items in the queue and try again later.

```js
  function doPost(endpoint, subQueue){
    if(subQueue.length === 0){
      idCounter = 0;
      return;
    }
    var preProcessQueue = endpoint === 'track' ? preProcessTrackQueue : preProcessEngageQueue;
    var queueEncoded = base64.encode(JSON.stringify(preProcessQueue(subQueue)));

    $http.post(TRACKING_ENDPOINT+endpoint+'/', {data: queueEncoded}, {
      headers: {'Content-Type': 'application/x-www-form-urlencoded'},
      transformRequest: function(obj) {
        var str = [];
        for(var p in obj) {
          str.push(p + "=" + obj[p]);
        }
        return str.join("&");
      }

    }).then(function pushSuccess(){
      removeQueueItems(subQueue);
      schedulePush();

    }, function pushFail(){
      schedulePush();
    });
  }

```

### Closing time? Save the queue for later

We need to persist the queue when the user switches app. This is because we don't want to loose all the things in the queue when the user switches or even closes the app. We can use [localStorage](https://developer.mozilla.org/en/docs/Web/API/Window/localStorage) to save the data. Every time the user switches or closes the app, we save the queue onto local storage. Then when the user opens the app again we restore the queue from local storage and continue periodically processing the queue.

```js
  window.document.addEventListener('pause', function(){
    persist(QUEUE, queueBuffer);
    queueBuffer.length = 0;
  }, false);

  window.document.addEventListener('resume', function(){
    var queue = restore(QUEUE);
    if(!queue){
      queue = [];
      persist(QUEUE, queue);
    }
    queueBuffer = queue;
    schedulePush();
  }, false);
```

### Queue getting long? Compress it

Because we are using local storage, we are limited by the amount of data we can store. Typically, local storage has a maximum budget of about 5mb. If we use local storage for our app data, this can be a problem. We have a few options, but for maximum compatibility with all browsers, we can still store the queue in local storage but compress it first. We use a string compressor to reduce the size of the string we store in local storage.

```js
  function persist(key, value){
    var valueCompressed = LZString.compress(JSON.stringify(value));
    window.localStorage.setItem(key, valueCompressed);
  }

  function restore(key){
    var item = window.localStorage.getItem(key);
    if(item){
      return JSON.parse(LZString.decompress(item));
    } else {
      return undefined;
    }
  }
```
As shown above, we use the LZString library to compress and decompress the stored string. As a very simple analysis, let's say we want to compress a few very simple mixpanel events which don't hold much data. The JSON data to store in localStorage would look like this:

```json
[{"event": "Level Complete", "properties": {"Level Number": 9, "distinct_id": "13793", "token": "e3bc4100330c35722740fb8c6f5abddc", "time": 1358208000, "ip": "203.0.113.9"}},
{"event": "Level Complete", "properties": {"Level Number": 9, "distinct_id": "13793", "token": "e3bc4100330c35722740fb8c6f5abddc", "time": 1358208000, "ip": "203.0.113.9"}},
{"event": "Level Complete", "properties": {"Level Number": 9, "distinct_id": "13793", "token": "e3bc4100330c35722740fb8c6f5abddc", "time": 1358208000, "ip": "203.0.113.9"}},
{"event": "Level Complete", "properties": {"Level Number": 9, "distinct_id": "13793", "token": "e3bc4100330c35722740fb8c6f5abddc", "time": 1358208000, "ip": "203.0.113.9"}}]
```

This is approximately 1392 bytes. If we compress it with LZString, it would look like this:

```
86 36 44 f0 60 0a 10 6e 02 76 01 e6 00 70 01 8c
84 96 62 03 08 83 60 0f 80 2d d8 0e 04 4f 03 60
89 46 38 01 44 12 c0 f7 84 25 3b 03 b8 22 cb 98
1c 80 57 80 00 22 58 8c 09 f0 4c cb 13 00 9d 36
c0 da 6f 0c 3e 00 39 9b c0 3c 60 04 c0 0c 8a 1d
9a 7e e0 69 0d 10 07 67 7d 44 54 62 65 01 01 d0
fe 85 2e fd e8 57 c8 0a 13 60 a1 3f 8b a3 19 80
80 98 0a 07 1b 80 4f 28 21 80 9c 98 8a 9c 98 99
11 3b 2a 15 8f 01 7f 84 84 8b 8b 5b 9b ac 0e 09
7e be 1d 80 a5 0b 81 6e 54 a5 00 98 93 2f 38 6d
02 16 bf 0e 3e 0e 19 31 15 05 83 2c 0b 13 17 3b
1f 0f 10 16 84 a8 34 bd 82 ac 9b 92 86 2a 8e 96
b1 81 ac a9 b5 85 2d ac 93 bd bb ab b7 a7 a0 9f
58 70 4c 64 62 5c 6a 72 db 76 4f 26 5e 0e 51 41
59 49 18 6a 75 05 7d 6d 4b 63 a9 4d 82 01 be 74
5c dd 14 21 44 8e a5 a1 cc 86 07 56 8a 1b 98 84
50 08 71 22 95 24 23 03 2d 42 6a 94 36 4d e1 8b
31 b1 76 a4 2f 36 81 9d e3 cc 78 70 be bc 90 00
2e 44 8a 12 12 c4 14 49 31 a9 91 e5 bd 40 85 f2
07 37 ae dc f9 e5 aa d4 d6 00 18 af 04 17 82 41
bd a6 40 28 18 d6 85 c7 91 8c f0 92 26 5a 89 69
63 43 8a e4 6a 3c 96 d0 25 86 cc 6d 72 56 12 5a
71 98 9d a4 e7 e9 55 26 9b d5 e4 70 9e 3c 5f d9
df 2b 53 9c 94 42 55 85 a8 1a ac a1 68 d2 74 01
00 80
```
which is approximately 193 bytes. That's a 14% reduction. We can therefore afford to store more events to be stored.

## Show me the code!
It would be an interesting project to create a stand-alone JavaScript library, but I think it would be best to contribute it to [Mixpanel's open-source library](https://github.com/mixpanel/mixpanel-js) so that their JavaScript library supports it out of the box. Instead, this blog post was about how to implement it yourself - to show that there's not a lot going on under the hood. In fact, if you read the mixpanel library, you'll see that it really isn't that complicated.

However, if you'd like to see a working implementation of the above snippets, do have a look at the Angular service I created for my prayer times app. This can be found [here](https://github.com/meltuhamy/belfastsalah/blob/master/www/js/svc/mixpanel.js).

