# gulp-endpoint

Sometimes we want to pipe files from a web server into Gulp.

And sometimes we want to pipe files out of Gulp into a web server.

But sometimes we just want Gulp to actually *be* a web server.

# Overview

Look at this:

```javascript
gulp.task('abc', function() {

});

gulp.task('xyz', function() {

});
```

Doesn't that look a little bit like an API?

I spent a lot of time looking at Sails, Hapi, LoopBack and more, and they're all really great. But I also realised that I was putting a lot of energy into turning my apps into streams. One minute it's a static site generator, the next an email pipeline using Handlebars templates or a transformation pipeline extracting text from PDFs...or even just simply pushing messages to queues -- I find that I'm always looking to use streams when I can.

And then I figured that many of these pipelines would work just as well in an API. Why not just have one set of Gulp tasks for doing all my template transformation work, and then use those tasks both to send email and to run in Express?

It seems simple enough -- the answer seemed to be to wrap Gulp so that I could expose its tasks as Express endpoints.

# Usage

To use simply install `gulp-endpoint`:

```shell
npm install --save gulp-endpoint
```

and then run `endpoint` from the command-line:

```shell
endpoint
```

This will load your `gulpfile`, set up a route for each task, and then launch a server on the default port (3000).

## Options

### Port (default is 3000)

To select another port to listen on, either define the `PORT` environment variable:

```shell
PORT=3005 endpoint
```

or use the `--port` command line option:

```shell
endpoint --port 3005
```

# Behaviour

Any request that hits a URL with the same name as the Gulp task will cause the task to be run. The `gulp.src` and `gulp.dest` functions are patched so that data provided with a `POST` to the task will become a source file, and any data at the end will be returned to the API caller.

# Example

For example, say we have the following simple `gulpfile`:

```javascript
var gulp = require('gulp');
var concat = require('gulp-concat');
var swig = require('gulp-swig');
var frontMatter = require('gulp-front-matter');

gulp.task('concat', function() {
  return gulp.src(['hello.txt', 'world.txt'])
    .pipe(concat('result.txt', {newLine: ' '}))
    .pipe(gulp.dest('output'));
});

gulp.task('compile-page', function() {
  return gulp.src('page.html')
    .pipe(frontMatter({property: 'data'}))
    .pipe(swig())
    .pipe(gulp.dest('build'));
});

gulp.task('default', ['compile-page']);
```

Running `endpoint` from the same directory as this file will result in three endpoints being created:

```
http://localhost:3000/
http://localhost:3000/compile-page
http://localhost:3000/concat
```

The first two both refer to the `compile-page` task, since any task called 'default' is connected to the root.

If a `GET` request is made to any of these URLs then the task is run with the usual `gulp.src` function in place, which means processing of any values set in the `gulpfile` will take place. The results of the operations will be returned in the HTTP response.

However, if a `POST` request is made then the document that is posted as part of the request is fed in to the `src` stage.
