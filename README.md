# AngularJS Template, by David Huertas

## Introduction
This **Angular JS Template** is a good start point for a new AngularJS project with **Gulp** as the task manager and **Bower** as the front-end dependencies manager.
Most information of this guide is taken from Michael Herman's website, http://mherman.org/blog/2014/08/14/kickstarting-angular-with-gulp. Although, there where a few mistakes that I have corrected in this version, like the content of the _.bowerrc_ file or the _NPM Local Dependencies_ that were missing.

## Content

 - Project Setup
 - Develop a Sample App
 - Create the build
 - Conclusion
 
## Project Setup
### Install Dependencies

#### Setup a project folder and create a  _package.json_  file:

```
$ mkdir project_name && cd project_name
$ npm init
```
The  `npm init`  command helps you create your project’s base configuration through an interactive prompt. Be sure to update the ‘entry point’ to ‘gulpfile.js’. You can just accept the defaults on the remaining prompts.

Do the same for Bower:
```
$ bower init
```

Accept all the defaults. After the file is created update the  `ignore`  list:

```
"ignore": [
  "**/.*",
  "node_modules",
  "app/bower_components",
  "test",
  "tests"
],
```

#### Install global dependencies:

```
$ npm install -g gulp bower
```

#### Bower install directory

You can specify where you want the dependencies (commonly known as bower components) installed to by adding a  _.bowerrc_  file and adding the following code:

```
{
  "directory": "app/bower_components"
}
```

#### Install local dependencies:

_NPM_

```
$ npm install gulp bower jshint run-sequence gulp-clean gulp-jshint gulp-uglify gulp-minify-css gulp-connect --save
```

_Bower_

```
$bower install angular angular-animate angular-route jquery animate.css bootstrap fontawesome --save
```

> The  `--save`  flag adds the dependencies to the  _package.json_  and  _bower.json_  files, respectively.

We’ll address each of these dependencies shortly. For now, be sure you understand the project’s core dependencies:

-   **[Gulp](http://gulpjs.com/)**  is a Javascript task runner, used to automate repetitive tasks (i.e., minifying, linting, testing, building, compiling) to simplify the build process.
-   **[Bower](http://bower.io/)**  manages front-end dependencies.

### Folder Structure

Let’s setup a base folder structure:

```
.
├── app
│   ├── bower_components
│   ├── css
│   │    └── main.css
│   ├── img
│   ├── index.html
│   ├── partials
│   │    ├── partial1.html
│   │    └── partial2.html
│   └── js
│   │    └── main.js
├── .bowerrc
├── .gitignore
├── bower.json
├── gulpfile.js
├── node_modules
└── package.json
```

Add the files and folders not already included. This structure is based on the popular  [Angular Seed](https://github.com/angular/angular-seed)  boilerplate, developed by the Angular team.

### Gulp

To start, we just need the following code:

```
// gulp
var gulp = require('gulp');

// plugins
var connect = require('gulp-connect');


gulp.task('connect', function () {
  connect.server({
    root: 'app/',
    port: 8888
  });
});
```

This allows us to serve our future Angular app on a development server running on port 8888.

### Test

Let’s test it out. Add the word ‘hi’ to the  _index.html_  file, then run the following command:

```
$ gulp connect
```

Navigate to  [http://localhost:8888/](http://localhost:8888/)  and you should see ‘hi’ staring back at you. Let’s build a quick sample app. Keep the server running…

## Develop a Sample App

### _index.html_

```
<!DOCTYPE html>
<html ng-app="SampleApp">
  <head lang="en">
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="">
    <meta name="author" content="">
    <title>Angular-Gulp-Browserify-Starter</title>
    <!-- styles -->
    <link rel="stylesheet" href="bower_components/bootstrap/dist/css/bootstrap.css"/>
    <link rel="stylesheet" href="bower_components/fontawesome/css/font-awesome.css"/>
    <link rel="stylesheet" href="bower_components/animate.css/animate.css"/>
    <link rel="stylesheet" href="css/main.css"/>
  </head>
  <body>
    <div class="container">
      <h1>Angular-Gulp-Browserify-Starter</h1>
      <!-- views -->
      <div ng-view></div>
    </div>
    <!-- scripts -->
    <script src="bower_components/jquery/dist/jquery.js"></script>
    <script src="bower_components/angular/angular.js"></script>
    <script src="bower_components/angular-route/angular-route.js"></script>
    <script src="bower_components/angular-animate/angular-animate.js"></script>
    <script src="bower_components/bootstrap/dist/js/bootstrap.js"></script>
    <script src="js/main.js"></script>
  </body>
</html>
```

This should look familiar. The  `ng-app`  directive initiates an Angular app while  `ng-view`  sets the stage for routing.

### _main.js_

```
(function () {

'use strict';


  angular.module('SampleApp', ['ngRoute', 'ngAnimate'])

  .config([
    '$locationProvider',
    '$routeProvider',
    function($locationProvider, $routeProvider) {
      $locationProvider.hashPrefix('!');
      // routes
      $routeProvider
        .when("/", {
          templateUrl: "./partials/partial1.html",
          controller: "MainController"
        })
        .otherwise({
           redirectTo: '/'
        });
    }
  ]);

  //Load controller
  angular.module('SampleApp')

  .controller('MainController', [
    '$scope',
    function($scope) {
      $scope.test = "Testing...";
    }
  ]);

}());
```

Again, this should be relatively straightforward. We setup the basic Angular code to establish a route handler along with a controller that passes the variable  `test`  to the template.

### _partial1.html_

Now let’s add the partial template:

```
<p>{{ test }}</p>
```

### Test

Back in your browser, refresh the page. You should see the text:

```
Angular-Gulp-Browserify-Starter

Testing...
```

## Create the Build

Now that our app is working locally, let’s modify our  _gulpfile.js_  to generate a deployable build. Kill the server.

```
// gulp
var gulp = require('gulp');

// plugins
var connect = require('gulp-connect');
var jshint = require('gulp-jshint');
var uglify = require('gulp-uglify');
var minifyCSS = require('gulp-minify-css');
var clean = require('gulp-clean');
var runSequence = require('run-sequence');

// tasks
gulp.task('lint', function() {
  gulp.src(['./app/**/*.js', '!./app/bower_components/**'])
    .pipe(jshint())
    .pipe(jshint.reporter('default'))
    .pipe(jshint.reporter('fail'));
});
gulp.task('clean', function() {
    gulp.src('./dist/*')
      .pipe(clean({force: true}));
});
gulp.task('minify-css', function() {
  var opts = {comments:true,spare:true};
  gulp.src(['./app/**/*.css', '!./app/bower_components/**'])
    .pipe(minifyCSS(opts))
    .pipe(gulp.dest('./dist/'))
});
gulp.task('minify-js', function() {
  gulp.src(['./app/**/*.js', '!./app/bower_components/**'])
    .pipe(uglify({
      // inSourceMap:
      // outSourceMap: "app.js.map"
    }))
    .pipe(gulp.dest('./dist/'))
});
gulp.task('copy-bower-components', function () {
  gulp.src('./app/bower_components/**')
    .pipe(gulp.dest('dist/bower_components'));
});
gulp.task('copy-html-files', function () {
  gulp.src('./app/**/*.html')
    .pipe(gulp.dest('dist/'));
});
gulp.task('connect', function () {
  connect.server({
    root: 'app/',
    port: 8888
  });
});
gulp.task('connectDist', function () {
  connect.server({
    root: 'dist/',
    port: 9999
  });
});


// default task
gulp.task('default',
  ['lint', 'connect']
);
gulp.task('build', function() {
  runSequence(
    ['clean'],
    ['lint', 'minify-css', 'minify-js', 'copy-html-files', 'copy-bower-components', 'connectDist']
  );
});
```

**What’s happening here?**

1.  [gulp-jshint](https://github.com/spenceralger/gulp-jshint)  checks for code quality in the JS files. If there are any issues the build fails and all errors output to the console.
2.  [gulp-clean](https://github.com/peter-vilja/gulp-clean)  removes the entire build folder so that we start fresh every time we generate a new build.
3.  [gulp-uglify](https://github.com/terinjokes/gulp-uglify)  and  [gulp-minify-css](https://www.npmjs.com/package/gulp-minify-css)  minify JS and CSS, respectively.

### Build commands

**Default**

The default task,  `gulp`, is a compound task that runs both the  `lint`  and  `connect`  tasks. Again, this just serves the files in the “app” folder on  [http://localhost:8888/](http://localhost:8888/).

**Build**

The build task creates a new directory called “dist”, runs the linter, minifies the CSS and JS files, and copies all the HTML files and Bower Components. You can then see what the final build looks like on  [http://localhost:9999/](http://localhost:9999/)  before deployment. You should also run the  `clean`  task before you generate a build.

Test this out:

```
$ gulp build
```

## Conclusion

Well, hopefully you now have a better understanding of how Gulp can greatly simply the build process, handling a number of repetitive tasks. Next time we’ll clean up some of the mess that the Bower components leave behind by adding Browserify into the mix and detail a nice workflow that you can use for all your Angular projects.