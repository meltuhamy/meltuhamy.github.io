---
title: 'Angular and Optimizely: A/B testing on SPAs'
tags:
  - ab testing
  - angular
  - blinkbox
  - optimizely
  - spa
id: 52
categories:
  - dev
date: 2015-03-05 12:08:23
---

One of my first tasks at [blinkbox books](https://www.blinkboxbooks.com/) was to integrate [Optimizely](https://www.optimizely.com/)'s powerful A/B testing service with our dynamic Angular JS single page web app. This blog post describes my experience and also shows how this can be done in a way that eliminates [flashes of unstyled content (fooc)](http://en.wikipedia.org/wiki/Flash_of_unstyled_content).

### How Optimizely works

![Optimizely's WYSIWYG editor (screenshot from optimizely.com)](https://help.optimizely.com/hc/en-us/article_attachments/201205034/Screen_Shot_2012-11-20_at_1.27.34_PM.png)

Optimizely is a [A/B testing](https://www.optimizely.com/ab-testing/) service that allows creating one or more variations of a page, and targeting those variations on a percentage of the page's visitors. The way it works is simple. Optimizely provides a WYSIWYG interface which allows a user (e.g. the marketing team) to make changes to a variation. The changes translate into some jQuery code. This code is embedded into a JavaScript snippet that is inserted into the page.
[![How Optimizley changes the page](http://meltuhamy.com/wp-content/uploads/2015/03/optimizely-flow-1024x244.png)](http://meltuhamy.com/wp-content/uploads/2015/03/optimizely-flow.png)

### Dynamically updating dynamic single page ecommerce websites

Optimizely works great if the page is rendered on the server, and user interactions are simply page changes. In other words, the jQuery snippet that changes the page would be applied once the page is loaded.

But what if the page loads a single page application (SPA)? In that case, Optimizely doesn't know if the page has changed, because that is controlled by JavaScript instead of page requests. This is explained in detail in the [support pages](https://help.optimizely.com/hc/en-us/articles/203326524-AngularJS-Backbone-js-and-other-Single-Page-Applications). To solve this, Optimizely provides an API that can be used to manually tell Optimizely that a page has changed. We simply call `optimizely.push('activate')` to tell Optimizely to activate any targeted by the current URL.

However, SPAs aren't just about changing pages dynamically. They can involve very dynamic views and components that asynchronously request content dynamically. For example, if the user scrolls down, more items can be dynamically requested and displayed. To make use of Optimizely's WYSIWYG editor, we needed to support _any_ update of _any_ part of the page at _any_ url. This would mean that we could pass Optimizely to the marketing team without worrying about making any kind of code change to make it work. This post is an overview of what I did to get Optimizely to correctly apply modifications to any part of a SPA.

### The naive solution: activate experiments when the page changes

There's a few events that we could hook into and activate Optimizely experiments. Each of these solutions fixes a problem but introduces another problem. Nevertheless, let's talk through each of them.

*  [$routeChangeSuccess](https://docs.angularjs.org/api/ngRoute/service/$route): The obvious one. Every time the page changes, we need to apply any experiments that might be associated with the new URL. We call `optimizely.push('activate')` and this will take care of everything for us. The problem with this is that there might be angular directives and embedded modules in the page which load dynamically. In this case, the Optimizely snippet would be applied, but no actual changes would be made because the required elements would not be loaded yet.

*   [$includeContentLoaded](https://docs.angularjs.org/api/ng/directive/ngInclude): Handling dynamic templates. To fix the problem above, we could listen for this event and call `optimizely.push('activate')` every time it fires. This will work but only for the first time that the template is loaded. For example, if we load a template for one 'tab', then switch tabs, the second time the user visits the first tab $includeContentLoaded will _not_ be fired, as it is already loaded and cached by angular. The other problem is angular directives that have external templates will not trigger this event when their templates are loaded.

*   [$browser.notifyWhenNoOutstandingRequests](https://github.com/angular/angular.js/blob/master/src/ng/browser.js#L75): An estimate of when the page has finished 'loading'. This private API is what is used by [protractor](https://github.com/angular/protractor) for end-to-end tests. If we register a callback that activates Optimizely when the page has finished loading according to angular, this would apply any page modifications correctly to the page. However, the drawback is that it will take some time for the page to finish loading and so there will be a very obvious flash of un-styled content.

### A more comprehensive solution: hook into the digest cycle.

What if we knew roughly when the DOM has changed by Angular? We could then apply the Optimizely changes every time Angular changed the DOM. Luckily, we do know when the DOM has changed - in the [digest](http://angular-tips.com/blog/2013/08/watch-how-the-apply-runs-a-digest/) cycle. We can simply call the activate method every time angular executes the digest cycle. And this is easily done by setting up an infinite watch, like below:

```js
var force = true;
$rootScope.$watch(function () {
  setTimeout(function () {
    force = !force;
  });
  return force;
}, function () {
  $window.optimizely.push(['activate']);
});
```


I found that having this solution along with listening for the $routeChangeSuccess event worked best, but there are still some problems...

#### Applying an experiment multiple times.

One could ask, surely we're calling the Optimizely API dozens of times. Is this wise? Well, it turns out that the Optimizely snippets are idempotent - meaning if we call it multiple times, it won't change the page multiple times.

![Too many XHR requests](http://meltuhamy.com/wp-content/uploads/2015/03/xhr-toodamnhigh.jpg)

However, I did notice that several XHR requests ended up being called because of Optimizely's logging feature. This is bad. Every time we call `optimizely.push('activate')`, an XHR request is queued. This is not only bad network usage, it will also drain the battery and is just pure evil. We had to have a workaround. It would be nice if Optimizely allowed us to disable logging for a single page, but until then, I implemented an incredibly hacky workaround. The solution: monkey-patch the XHR.

```js
function patchXHR() {
  var originalOpen = window.XMLHttpRequest.prototype.open;
  var originalSend = window.XMLHttpRequest.prototype.send;

  var prevUrl;
  window.XMLHttpRequest.prototype.open = function (type, uri) {
    if (uri.lastIndexOf('log.optimizely.com') >= 0 && fromDigest){
      // we set this up in order to intercept the request in the 'send' function.
      this._requestURI = uri;
      fromDigest = false;
    }
    originalOpen.apply(this, arguments);
  };

  window.XMLHttpRequest.prototype.send = function () {
    if (typeof this._requestURI === 'string'
        && this._requestURI.lastIndexOf('log.optimizely.com') >= 0) {
      var currentUrl = $location.path();
      if (currentUrl === prevUrl) {
        // we prevent the request if this was the same page
        this.abort();
      } else {
        // we allow requests on actual page changes.
        prevUrl = currentUrl;
        originalSend.apply(this, arguments);
      }
    } else {
      // we allow all non-optimizely requests
      originalSend.apply(this, arguments);
    }

  };
}
```

This just about solves the requests problem, but there are _still_ some other problems...

### Undoing an experiment

What if we had an experiment that removed the navigation bar, or the footer, or any other common element in our website? For example, what if we wanted to remove the page footer but only in the 'About Us' page, and not any other pages? Well, because we're using a modular single page application, once we remove the element (using the Optimizely snippet), it'll be removed from our whole application until the user reloads! This is because common elements like the page header and footer don't change between page changes. All the routing is done in JavaScript, remember?

Unfortunately, there wasn't any elegant way to fix this. We decided to simply not allow these types of changes unless they are app-wide. It would have been nice if there was a way of undoing a snippet, but this would definitely be a challenge (e.g. if you remove an element and not hide it, then re-insert it, how do you ensure angular still knows about it? What about memory and performance considerations).

### Conclusion

Getting third party services to work with a SPA is hard. Optimizely should invest engineering effort to make it work with at least the most popular frameworks such as Angular, or at least provide more fine grained APIs to allow users to integrate manually.

While the solution of using Angular's digest cycle worked, it isn't great and smells like a hack. There needs to be a better way of applying A/B testing experiments on a single page app, and this would require a lot of thought.

Having said that, for very simple web apps, the approach described above would probably be overkill. In fact, for most simple applications, using the $routeChangeSuccess approach would work just fine. However, if your app is dynamic and has many components and directives which are also dynamic, getting Optimizely to work will need a bit more hacking - and this article was supposed to be an overview of what we did at blinkbox to get it to work.

### Show me the code

To do all of the above (and a bit more), I wrote an Angular service for the [blinkbox books application](https://github.com/blinkboxbooks/client-web-app.js). You can see the service on [GitHub](https://github.com/blinkboxbooks/client-web-app.js/blob/master/app/scripts/services/optimizely_service.js). I'm also thinking about taking this out and making it its own service so anyone can drop it into their app.