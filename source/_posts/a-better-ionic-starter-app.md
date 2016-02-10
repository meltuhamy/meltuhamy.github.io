---
title: A better Ionic starter app
thumbnail: /images/ionitron-avatar.png
banner: /images/angularjs-ionic-framework.jpg
tags:
  - gulp
  - ionic
  - karma
id: 103
categories:
  - dev
date: 2015-03-22 12:32:12
---

_TLDR_: I wrote a nice ionic starter app that anyone can use as a boilerplate. [You can find it on GitHub](https://github.com/meltuhamy/ionic-base).

While I was writing [my first Ionic app](http://meltuhamy.com/tech/dev/ionic-speed-writing-a-prayer-times-smartphone-app-in-a-day/), I realised there are a lot of tools from front end web development that can be added to the project. The [default starter app](https://github.com/driftyco/ionic-starter-tabs) was a bit too simple.

### A better file structure

In the default starter app, every angular component was in its own file.

```
js
├── app.js
├── controllers.js
└── services.js
```

Opening up `controllers.js` will show all the controllers of our app. What if we had many? I prefer having a file for each controller.

A better starter app should have each controller, service, constant, etc. in its own file. That way, we can quickly get to the code we're looking for later on.

```
js
├── app.js
├── controllers
│   ├── account.ctrl.js
│   ├── chatdetail.ctrl.js
│   ├── chats.ctrl.js
│   └── dash.ctrl.js
└── services
    └── chats.service.js
```

The suffix in the names (`.ctrl.js`) is optional, but allows us to distinguish between controllers/services with the same name.

### Unit testing support with Karma

I was surprised to find that the default project didn't have unit test support. This was strange because Angular already had really good unit and end to end testing support by default. To fix this, we simply need to add and  `karma.conf.js`. Most of it is the default settings (simply run `karma init`, make sure you have [karma](http://karma-runner.github.io) installed), with the following included files:

```js
{
    files: [
        'www/lib/ionic/js/ionic.bundle.js',
        'node_modules/angular-mocks/angular-mocks.js',
        'www/lib/ngCordova/dist/ng-cordova.js',
        'www/lib/ngCordova/dist/ng-cordova-mocks.js',

        'test/**/*.test.js',
        'src/js/**/*.js'
    ]
}

```

Now, we can run `karma start` to run the unit tests. We can also update our package.json to include a testing step. This is useful when using travis for continuous builds.

```json
{
    "scripts": {
        "test": "./node_modules/karma/bin/karma start --single-run --browsers PhantomJS"
    }
}

```

Running `npm test` will run the unit tests.

### Concatenating, uglifiying and building our app

We could create an optimized app by putting everything in one file and reducing the number of requests (even if they are all local). To do this, we use a few gulp plugins.

First, we move all our _source_ files into a new folder called `src`. The plan is to combine all these source files into a single `app.js` file that will go in the `www` folder. Here's how we do it:

```js
gulp.task('build', function () {
  return gulp.src('src/js/**/*.js')
          .pipe(sourcemaps.init())
          .pipe(ngAnnotate({
            single_quotes: true
          }))
          .pipe(concat('app.js'))
          .pipe(uglify())
          .pipe(sourcemaps.write())
          .pipe(header('window.VERSION = "<%= pkg.version %>;";', { pkg : pkg } ))
          .pipe(gulp.dest('www/dist'));
});
```

The gulp task is easy to read, but there's a few things we didn't mention:

*   The `sourcemaps` plugin. Here, we write the sourcemap to the same source file. This will allow us to debug the files more naturally in chrome developer tools, even though they are uglified and concatenated into a single file.
*   The `ngAnnotate` plugin. We use this to allow us to minify Angular shorthand injections. E.g. `app.controller('MyCtrl', function($scope){});` becomes `controller('MyCtrl', ['$scope', function($scope){}]);`
*   The `header` plugin. We use this to smartly insert the app's version number inside the app itself. We talk about versioning later in this article.
*   Finally, we write to the `www/dist` folder. Putting the output into another folder allows us to modify `.gitignore` so we don't version the generated files.

While we're here, we can also modify the `sass` gulp task so it also goes into the `www/dist` folder.

```js
gulp.task('sass', function(done) {
  gulp.src('src/scss/ionic.app.scss')
    .pipe(sass())
    .pipe(gulp.dest('./www/dist/css/'))
    .pipe(minifyCss({
      keepSpecialComments: 0
    }))
    .pipe(rename({ extname: '.min.css' }))
    .pipe(gulp.dest('./www/dist/css/'))
    .on('end', done);
});
```

#### Faster page changing using Angular's `$templateCache`

When switching between 'pages' in a SPA, new template file requests are made. If we move all these template files into a single file and pre-cache it, these requests can be saved. There's a gulp task for that.

```js
gulp.task('templates', function(){
  return gulp.src('src/templates/**/*.html')
    .pipe(templateCache('templates.js',{module: 'starter', root:'templates/'}))
    .pipe(gulp.dest('www/dist'));
});
```

#### Making it play nice with `ionic serve`

Finally, let's modify our `ionic.project` file to make use of the build steps.

```json
{
    "gulpStartupTasks": [
        "default",
        "watch"
    ]
}

```

This will run our `gulp` and `gulp watch` tasks which we create as follows, in our gulpfile.js:

```js
gulp.task('default', ['sass', 'templates', 'build']);

gulp.task('watch', function() {
  gulp.watch(paths.sass, ['sass']);
  gulp.watch(paths.js, ['build']);
  gulp.watch(paths.templates, ['templates']);
});
```

Great, now every time we make a change, our build will be triggered and the page will automatically refresh!

### Updating our app version

Finally, I noticed that every time I update my app version number, I have to update it in quite a few places. package.json, bower.json, config.xml, and anywhere I use it inside my actual app (e.g. in the 'about' page). Instead, it would be nice to do this once. Luckily there's a gulp task for that, and it's really simple:

```js
gulp.task('bump', require('gulp-cordova-bump'));
```

Now when I want to update my app version number, all I have to do is run one of the following:

```
$ gulp bump --patch
$ gulp bump --minor
$ gulp bump --major
$ gulp bump --setversion=2.1.0
```

And gulp will patch everything up. Oh, and remember that `banner` step in the build mentioned above? Well, this will get the version number from our package.json, put it into our compiled app.js script as a global variable (`window.VERSION`), and we can now use it in our app! To make it play nice with Angular, we can put it into our `$rootScope` so we can use it directly in our template. We simply add the following line in our `run` block:

```js
.run(function($ionicPlatform, $rootScope) {
  $rootScope.VERSION = window.VERSION;
  // ...
}
```

and we can use it in any template:

```html
<div>
  App Version {{VERSION}}
</div>
```

### Show me the code

Feel free to work with it on GitHub:
[https://github.com/meltuhamy/ionic-base](https://github.com/meltuhamy/ionic-base).

### Credits
Post thumbnail and image are from the Ionic project.