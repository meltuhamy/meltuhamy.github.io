---
title: 'Ionic Speed: Writing a prayer times smartphone app in a day'
tags:
  - android
  - app
  - ionic
  - ios
  - prayer
id: 81
categories:
  - dev
date: 2015-03-11 14:09:54
---

_Update:_ I'm happy to announce the app has been released on the Google Play Store. [Check it out!](http://goo.gl/EhJdx1)

Whilst I was in Belfast, I promised my sister I'd finally write an app for them to show our Mosque's prayer times. Initially I planned to use this as a chance to learn some new tech such as native iOS and Android development, but unfortunately I kept procrastinating and didn't manage to do it. With a couple of days remaining before leaving Belfast, I realised I had not fulfilled my promise, and decided: what's the fastest way to write an application that _works_ and looks nice - given my current skill set? The answer was obvious. [Ionic Framework](http://ionicframework.com/).

Ionic Framework as an obvious pick because of the following reasons:

*   I already have experience writing Angular apps. I could jump right in and know what I'm doing.
*   The app I'm making is really simple, and I already have an idea of how it'll work.
*   I want my app to work on iOS and Android, and I don't have the time to write different apps for each platform. Ionic works this out for you ;)

So going ahead with Ionic, I set it up straight away.

### Wait, what are you building again?

Muslims pray five times a day. The times of prayer is based on the position of the sun in the location you're in. While you can work out the prayer times [using some math](http://praytimes.org/calculation/) (and in fact there are hundreds of apps that already do this), there is an added importance of praying at the same time as other Muslims in the area. So if everyone in the city used their own app that works it out, each person would be praying at different times! The solution was to use the prayer times at the Mosque that's closest to you. Fortunately in Belfast there are only two mosques, and they use the same prayer timetable. So, I decided to write an app that uses the mosque's prayer times.

![A screenshot of the app in action](/images/salahtimes-screenshot-614x1024.png)

### Setting up

![So many tech. So little effort to set up.](/images/ionic-tools.png)

I followed the normal set up procedure without worrying too much about the details. It really is [this easy](http://ionicframework.com/getting-started/):

```
    npm install -g cordova ionic
    ionic start belfastsalah tabs
    cd belfastsalah
    ionic platform add ios
    ionic build ios
    ionic emulate ios
```

### App structure

By default, the Ionic app structure looked something like this:

*   `app.js`
*   `controllers.js`
*   `services.js`

And each file contained all the angular components of our app. I prefer structuring the app differently, so each service, controller, and so on is in its own file.


*   `app.js`: Contains config and module definitions.
*   `constants/`: Inside this folder, all our angular constants (in our case, our prayer time data) will go here.
*   `controllers/`: Each controller will live in its own file in this folder
*   `filters/`: same for filters
*   `services/`: and services

Now that we have a much better structure, we can now think about the services, controllers, constants and filters we need. Luckily, I already had all the data for the prayer time table. It's essentially a JSON array containing objects for each day in the year - so for 366 days we had 366 objects. Each object is an array representing the different prayer times of that day. For example, here's what 1 January's prayer times looks like:

```js
["1","1","06:49","08:44","12:29","13:55",null,"16:11","18:00"]
```

This file would go in as an [angular constant](https://docs.angularjs.org/api/auto/service/$provide#constant), so we can inject it as a dependency wherever we need it.

Now, we define our services. We need a service that keeps track of the time ticking, I called it the `Ticker` service. We also need a service that gets the required prayer times for a day or a month. I called it `PrayerTimes`. We define and implement a few methods: `getByDate` would get the prayer times given a JavaScript [Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date) object. `getByMonth` would get the prayer times for a whole month, and `getNextPrayer` would get the next and previous prayers and their times, given a JavaScript Date object.

Finally, we define our controllers. We essentially define a controller for each tab. We therefore have three: `Today`, `Month`, and `Settings`. The `Today` controller would make use of the `Ticker` and `PrayerTimes` services to update the remaining time for the next prayer. The `Month` controller would simply display all the times for the current month, and the `Settings` controller is a work in progress, but would allow disabling and enabling app notifications (see conclusion).

### Adding some goodies to help us out

To save time and effort, I used [lodash](https://lodash.com/), [moment.js](http://momentjs.com/), and [angular-moment](https://github.com/urish/angular-moment) to help with array searching, time calculations, and displaying times as strings in the view. To do this with Ionic, you can simply type `ionic add &lt;package name&gt;` and Ionic will take care of it (uses bower).

### Conclusion and source code

Overall, it was incredibly easy and fast to get the app finished. In fact, Ionic allowed me to also test the app without having to connect a device using their [Ionic View](http://view.ionic.io/) service. Really, the only thing left is easy deployment / integration with Google Play Store / AppStore, though I'm not sure if that's even possible.

There are a few things left, however. I didn't get time to set up notifications for prayers, but this is certainly feasible using the [localNotification](https://github.com/katzer/cordova-plugin-local-notifications) cordova plugin. Hopefully I can get this done in a future version!

Check out the [source code on GitHub](https://github.com/meltuhamy/belfastsalah).