---
layout: post
title: "It is all about streams"
date: 2014-04-20 12:27:30 -0300
comments: true
categories: [Node.js, Gulp.js]
---
{% youtube aQyXeLSL0II %}

For Kramer it was all about *levels*, for us it is all about *streams*. I bumped into [Gulp.js](http://gulpjs.com/) not so long ago, more specifically when I started working on a Yeoman generator project for my job.
I needed to create a new generator to scaffold a Javascript ecommerce application we use, which was very hard to deploy.

So I created this Yeoman generator and when I had to decide whether to use [Grunt.js](http://gruntjs.com/) or Gulp, I tried them both but I felt in love with Gulp.js right away. 

For starters, Gulp concept is way more superior than Grunt, in terms of how tasks are being processed. In Grunt you just create regular tasks using javascript, which is cool but with Gulp gets the best out of node.js by using pure *streams*. 

And here's why I chose Kramer to start this post with, it is not about *levels* but about *streams*, although in a sense, even when pipes is a concept that comes from the old Unix world, it is not that easy to grasp.
However, once you discover it, you'll want to do everything with pipes.

## What are streams?

As we know, at the [node.js](http://nodejs.org/) world, every interaction with disk and network involves passing callbacks to functions. When we think of reading and processing a file, we think of something like this:

    var http = require('http');
    var fs = require('fs');
    var server = http.createServer(function (req, res) {
        fs.readFile(__dirname + '/myfile.txt', function (err, data) {
            res.end(data);
        });
    });
    server.listen(8000);

This code implies that the whole file will have to be buffered into memory before it starts being sent to the client. This would be our first approach to solve this, however it is clearly not an ideal solution in terms of performance.

However, Node.js provides us with a more efficient way to deal with this scenario: using something called streams. Let's take a look at this code:

    var http = require('http');
    var fs = require('fs');
    var server = http.createServer(function (req, res) {
        var stream = fs.createReadStream(__dirname + '/myfile.txt');
        stream.pipe(res);
    });
    server.listen(8000);

In this example, we create a reader stream and it is the *pipe* function which takes care of listening and sending chunks of this file immediately after the file is read from the disk.

Besides, the use of *pipe* provides us with an extra benefit, it handles buffering efficiently so no data will be sent to the client if the client is not ready to process it, (probably due to a slow connection network or high-latency issues).

Now we can see streams are useful but how can we use them and most importantly, what can Gulp.js do for us?

Let's take a brief tour on the streams node.js has to offer.

Streams are connected using the pipe() function we have just seen, so just as it worked with Unix in the old days, chaining commands like this 

    ls | grep "setup_" | wc -l

works in node.js like this

.pipe(task1)
.pipe(task2)
.pipe(task3)

**FYI:** What the previous Unix command does is to list the directory content, send the result to a grep command to filter those entries with the "setup_" pattern in their name and finally counting the resulting lines.

### Types of streams

There are 5 types of streams: readable, writable, transform, duplex and classic but we are going to briefly cover just the first four ones in this post.

### Readable

Readable streams create data that can be consumed by other streams like writable, transform or duplex streams.

In order to create a readable stream we can do something like this:

    var Readable = require('stream').Readable;
    var rs = new Readable;
    rs.push('hello ');
    rs.push('world\n');
    rs.push(null);
    rs.pipe(process.stdout);

    $ node myreadstream.js
    hello world

We define a readable stream, push some content ending with *null*, (which is very important since this is what triggers the *end* event) and finally we pipe the result to the next stream. 
In this case we use *process.stdout* to display the result directly to the console (standard output).

In the previous example, we are pushing content directly but we could use the *_read* function to push chunks on-demand, which means only send data when the client asks for it. 

### Writable

This a stream for writing, which means you can *pipe* but you cannot use *src.pipe(writableStream)*

Let's see an example of it

    var Writable = require('stream').Writable,
    
    var ws = Writable();
    ws._write = function(chunk, enc, next) {
        var message = {
            timestamp: new Date().getTime(),
            message: chunk.toString('utf8')
        }
        console.dir(message);
        next();
    };
    process.stdin.pipe(ws);

    echo "This is a log message" | node mywritestream.js

In this example we create a *Writer* stream that could be used as a logger. *Writer* streams use the *write* function which receives the following parameters:

* chunk = What the *Reader* stream sends 
* encoding = Very important when using strings
* next(err) = It is the callback that tells the consumer that they can write more data. An optional *err* object parameter can be passed which emits an error event to the stream instance.

### Transform

Transform streams better known as *through* streams are the ones that allow read and write data. They are mainly used to read data, transform it somehow and write it out.

A good example could be a stream for reversing sentences.

    var stream = require('stream'),
        util = require('util');
    var Transform = stream.Transform;
    var Reverse = function(options) {
        Transform.call(this, options);
        this._reversedText = ''; 
    }
    util.inherits(Reverse, Transform);
    Reverse.prototype._transform = function(chunk, encoding, callback) {
        var text = chunk.toString('utf8');
        this._reversedText = text.split(' ').reverse().join(' ') + this._reversedText;
        callback();
    }
    Reverse.prototype._flush = function(callback) {
        this.push(this._reversedText, 'utf8');
        callback();
    }
    
    //Here's how we use this stream
    var reverseStream = new Reverse();
    reverseStream.on('data', function(data) {
        console.log(data.toString('utf8'));
    });
    reverseStream.write('Hello ');
    reverseStream.write('World ');
    reverseStream.end();

As we can see, we create a new Transform stream, declare a variable where we will store the modified text called: *_reversedText*.

Finally, the *transform* function is where all the magic happens. We transform the data into a String format, split the text by spaces, reverse the array and join it back as a string with spaces.

When the *flush* function is called the text is pushed.

In order to try this, we create a new instance of our stream and data is sent to standard output upon receiving it, check the *on* event.

### Duplex

These are special kind of *Transform* streams that allow bidirectional data flow. 

    a.pipe(b).pipe(a)

Chats are a good example when duplex streams could be used.

## What about Gulp.js?

So far we have written about streams but how can we use those streams?. 
Truth be told, although it is really fun to play with streams, most of the times you will probably find yourself digging into [npm](https://www.npmjs.org/) packages trying to find a gulp stream that solve your problems. And most of them times you will be successful at your search. 

### Installation

The first we have to do is to download [Gulp.js](http://gulpjs.com/) and install it.

    node install -g gulp 

### Tasks

Then, you have to create your *GulpFile.js* and start adding your tasks. 

In order to understand how tasks are defined, let's take a look at a GulpFile.js file.

Firstly, you call the needed requires according to the packages you need. In this example, you will need *gulp-concat and gulp-uglify*. 

***Important*** Make sure to download these packages before using them.

    npm install gulp-concat gulp-uglify

Then you can create your file like this

    var gulp = require('gulp');
    var concat = require('gulp-concat');
    var uglify = require('gulp-uglify');


In Gulp you define an origin, which is a directory where all files to be processed reside. Filters and wildcards can be used here, so in this case we would like to process all js files in every directory within the client directory.

    var paths = {
      scripts: ['client/js/**/*.js']
    };

Then, we define our tasks and chain our streams. In this example, we create a task called *scripts*, process all js files, send them to uglify(), concat the result into a new file called *all.min.js* and finally store them at the *build/js* directory (which is automatically created if it does not exist).

    gulp.task('scripts', function() {
      // Minify and copy all JavaScript (except vendor scripts)
      return gulp.src(paths.scripts)
        .pipe(uglify())
        .pipe(concat('all.min.js'))
        .pipe(gulp.dest('build/js'));
    });

We can create as many tasks as we like.

    // Another task
    gulp.task('another', function() {
      ...
    });

Finally we could even create a task that depends on previous tasks. What it does is to execute all referenced tasks.

    // The default task (called when you run `gulp` from cli)
    gulp.task('default', ['scripts', 'another']);

**Important: ** It is important to understand that in the last command, tasks are executed asynchronously, which means that *another* will not necessarily be executed *after* *scripts*. If we need to run dependent tasks in order, we must use another npm package called: [run-sequence](https://www.npmjs.org/package/run-sequence).

As I said, once you start with Gulp, you will see how many useful apps are available at [npm](https://www.npmjs.org/).

Last but not least, all you have to do to execute your tasks is this

    gulp default

Pretty awesome uh? 

Happy coding!







