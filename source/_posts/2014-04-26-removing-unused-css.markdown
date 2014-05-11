---
layout: post
title: "Removing unused CSS"
date: 2014-04-26 21:25:26 -0300
comments: true
categories: [Css, Node.js]
---

{% img left /images/unused-css.png 'unused-css' 'images' %}

It seems as if all web developers are lately aiming at reducing the size of the content delivered to the browser.

It is in fact a big challenge and optimizing a web site content delivery might imply a great deal of tweaks and tricks to be successfully accomplished.

Apache modules like [PageSpeed](http://www.modpagespeed.com) are vital in this mission. PageSpeed improves web page latency by modifying the resources on the page. For example a set of filters are applied to perform tasks such as altering the HTML content, changing CSS and JS files references or simply making pages to point to more optimized images.

Custom optimization strategies are applied to different assets in order to reduce the loading time. These ones involve inlining small resources, dynamically minifying JS and CSS files, and so on and so forth.   

These techniques can obviously be of great help but they are only the tip of the iceberg. Since we as developers have been starting using boilerplates with tools like Yeoman, we can build our own set of tools to automatically generate a better content before it is deployed to the web server.

This means that we now have a great deal of [npm](https://www.npmjs.org/) packages at our hands, ready to be piped with [Gulp.js](http://gulpjs.com/) ([please read my previous post about this](http://warambil.com/blog/2014/04/20/it-is-all-about-streams/)).

When I first heard about *Uncss* techniques I must confess I did not grasp the real potential behind it. At first, I thought it was just about removing unused css from static HTML pages. 

I thought that using [gulp-uncss](https://github.com/ben-eb/gulp-uncss) on static HTML files made no sense at all, specially considering that most of nowadays web sites are either server script generated pages or JavaScript SPAs (Single Page Applications).

It was not until I found Giacomo Martino's project ([Uncss](https://github.com/giakki/uncss)), that I could understand how awesome it is.

In order to fully understand how it all works, we must first learn about other apps.

## [PhantomJs](http://phantomjs.org)

PhantomJs is *headless Webkit scriptable with a Javascript API*. 
In other words, it is a full stack webkit where you can render any website as if it were rendered by a regular browser. 

For instance, you might need to use it for

* Headless website testing
* Screen capturing
* Page automation
* Network monitoring

Let's see a couple of examples taken from [their website](http://phantomjs.org/quick-start.html), to understand how it works. 

### Capture a web page

Create a very basic Javascript file named *Capture.js* 

    var page = require('webpage').create();
    page.open('http://example.com', function() {
      page.render('example.png');
      phantom.exit();
    });        

Then, all we do is this 

    phantomjs Capture.js

That's it. This script opens the indicated URL and renders the whole content into an example.png file. 

### Page loading time measurement

Create a javascript file called *LoadSpeed.js*

    var page = require('webpage').create(),
      system = require('system'),
      t, address;
    if (system.args.length === 1) {
      console.log('Usage: loadspeed.js <some URL>');
      phantom.exit();
    }
    t = Date.now();
    address = system.args[1];
    page.open(address, function(status) {
      if (status !== 'success') {
        console.log('FAIL to load the address');
      } else {
        t = Date.now() - t;
        console.log('Loading time ' + t + ' msec');
      }
      phantom.exit();
    });

Then, we must execute as follows

    phantomjs loadspeed.js http://www.google.com

and we should obtain an output like this

*Loading http://www.google.com Loading time 719 msec*

## So how does [Uncss](https://github.com/giakki/uncss) work then?

Basically, *UnCSS* removes unused CSS from your stylesheets by performing the following tasks:

1. PhantomJs loads HTML files and Javascript is executed
2. It checks what styles are actually being used in every page
3. A parser (css-parse) is applied to the concatenated bunch of css files 
4. All unused selectors are ruled out and an optimized resulting CSS file is created.

UnCss is a node.js project and you could [check its repository](https://github.com/giakki/uncss) if you would like to dive in and use it right away. 

However, we would like to go one step further.

## gulp-uncss

This GulpJs task uses UnCss to parse a set of *HTML* files and an original *CSS* file in order to remove unused styles. 

As any other gulp npm package this one is installed as follows

    npm install gulp-uncss --save-dev

Now, as usual we can pipe this task with the rest of our pre deployment tasks. I ran the following example against my Octopress blog.

    var gulp = require('gulp');
    var uncss = require('gulp-uncss');
    var minifyCSS = require('gulp-minify-css');
    gulp.task('default', function() {
        return gulp.src('/octopress/public/stylesheets/screen.css')
            .pipe(uncss({
                html: ['/octopress/public/index.html']
            }))
            .pipe(minifyCSS())
            .pipe(gulp.dest('./out'));
    });

As you can see, I also added a *minify* task to the process so you could see that this task can be piped to any other GulpJs task.

The overall process was very fast in this case and the original CSS file size was reduced from 38Kb to 12K.

## Further considerations

The task execution time can be drastically incremented if files are obtained directly from an external site, so it is recommended to run this task together with the rest of the optimization tasks you usually run with GulpJs on your */dist* directory, before your actual deployment.

If you need to run it against external generated HTML files, you should better previously save those files using PhantomJs and run the task locally (adding the local CSS file).

## Final thoughts

After digging into all these techniques, I feel more and more confident that GulpJs and node streams are the way to go as part of the development process.

[Yeoman](http://yeoman.io/) has proven to be a core foundation for our web projects and [Gulp.js](http://gulpjs.com/) is a *must* for batch tasks.

More GulpJs apps are being created everyday and we can easily add them to our GulpFile.js file in order to get a truly optimized bundle before deployment.

