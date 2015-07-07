# glob-fs [![NPM version](https://badge.fury.io/js/glob-fs.svg)](http://badge.fury.io/js/glob-fs)

> file globbing, for node.js.

## Usage

```js
var glob = require('glob-fs')({ gitignore: true });
var files = glob.readdirSync('**/*.js');
```

See [more examples](#examples) (WIP!):

* [glob.readdir](#async)
* glob.readPromise
* [glob.readStream](#stream)
* [glob.readdirSync](#sync)

## Table of contents

<!-- toc -->

* [Install](#install)
* [Usage](#usage)
* [API](#api)
* [Middleware](#middleware)
  - [Middleware examples](#middleware-examples)
  - [Middleware conventions](#middleware-conventions)
* [Globbing examples](#globbing-examples)
  - [async](#async)
  - [promise](#promise)
  - [stream](#stream)
  - [sync](#sync)
* [Events](#events)
  - [Event examples](#event-examples)
* [[node-glob] comparison](#-node-glob--comparison)
* [TODO](#todo)
* [Related projects](#related-projects)
* [Running tests](#running-tests)
* [Contributing](#contributing)
* [Author](#author)
* [License](#license)

_(Table of contents generated by [verb])_

<!-- tocstop -->

## Install

Install with [npm](https://www.npmjs.com/)

```sh
$ npm i glob-fs --save
```

## Usage

**Params**

All "read" methods take a glob pattern and an `options` object. Examples:

```js
// sync
var files = glob.readdirSync('*.js', {});

// async
glob.readdir('*.js', function(err, files) {
  console.log(files);
});

// stream
glob.readdirStream('*.js', {})
  .on('data', function(file) {
    console.log(file);
  });

// promise
glob.readdirPromise('*.js')
  .then(function(files) {
    console.log(file);
  });
```

## API

### [.readdir](lib/readers.js#L27)

Asynchronously glob files or directories that match the given `pattern`.

**Params**

* `pattern` **{String}**: Glob pattern
* `options` **{Object}**
* `cb` **{Function}**: Callback

**Example**

```js
var glob = require('glob-fs')({ gitignore: true });

glob.readdir('*.js', function (err, files) {
  //=> do stuff with `files`
});
```

### [.readdirSync](lib/readers.js#L59)

Synchronously glob files or directories that match the given `pattern`.

**Params**

* `pattern` **{String}**: Glob pattern
* `options` **{Object}**
* `returns` **{Array}**: Returns an array of files.

**Example**

```js
var glob = require('glob-fs')({ gitignore: true });

var files = glob.readdirSync('*.js');
//=> do stuff with `files`
```

### [.readdirStream](lib/readers.js#L89)

Stream files or directories that match the given glob `pattern`.

**Params**

* `pattern` **{String}**: Glob pattern
* `options` **{Object}**
* `returns` **{Stream}**

**Example**

```js
var glob = require('glob-fs')({ gitignore: true });

glob.readdirStream('*.js')
  .on('data', function (file) {
    console.log(file.path);
  })
  .on('error', console.error)
  .on('end', function () {
    console.log('end');
  });
```

### [Glob](index.js#L32)

Optionally create an instance of `Glob` with the given `options`.

**Params**

* `options` **{Object}**

**Example**

```js
var Glob = require('glob-fs').Glob;
var glob = new Glob();
```

### [.exclude](index.js#L156)

Thin wrapper around `.use()` for easily excluding files or directories that match the given `pattern`.

**Params**

* `pattern` **{String}**
* `options` **{Object}**

**Example**

```js
var gitignore = require('glob-fs-gitignore');
var dotfiles = require('glob-fs-dotfiles');
var glob = require('glob-fs')({ foo: true })
  .exclude(/\.foo$/)
  .exclude('*.bar')
  .exclude('*.baz');

var files = glob.readdirSync('**');
```

### [.use](index.js#L195)

Add a middleware to be called in the order defined.

**Params**

* `fn` **{Function}**
* `returns` **{Object}**: Returns the `Glob` instance, for chaining.

**Example**

```js
var gitignore = require('glob-fs-gitignore');
var dotfiles = require('glob-fs-dotfiles');
var glob = require('glob-fs')({ foo: true })
  .use(gitignore())
  .use(dotfiles());

var files = glob.readdirSync('*.js');
```

## Middleware

glob-fs uses middleware to add file matching and exclusion capabilities, or other features that may or may not eventually become core functionality.

**What is a middleware?**

A middleware is a function that "processes" files as they're read from the file system by glob-fs.

**What does "process" mean?**

Typically, it means one of the following:

1. matching a `file.path`, or
2. modifying a property on the `file` object, or
3. determining whether or not to continue recursing

### Middleware examples

**recursing**

Here is how a middleware might determine whether or not to recurse based on a glob pattern:

```js
var glob = require('glob-fs');

// this is already handled by glob-fs, but it 
// makes a good example
function recurse() {
  return function(file) {
    // `file.pattern` is an object with a `glob` (string) property
    file.recurse = file.pattern.glob.indexOf('**') !== -1;
    return file;
  }
}

// use the middleware
glob()
  .use(recurse())
  .readdir('**/*.js', function(err, files) {
    console.log(files);
  });
```

**exclusion**

Middleware for excluding file paths:

```js
// `notests` middleware to exclude any file in the `test` directory
function tests(options) {
  return function(file) {
    if (/^test\//.test(file.dirname)) {
      file.exclude = true;
    }
    return file;
  };
}

// usage
var glob = glob({ gitignore: true })
  .use(tests())

// get files
glob.readdirStream('**/*')
  .on('data', function (file) {
    console.log(file.path);
  })
```

### Middleware conventions

* **Naming**: any middleware published to npm should be prefixed with `glob-fs-`, as in: `glob-fs-dotfiles`.
* **Keywords**: please add `glob-fs` to the keywords array in package.json
* **Options**: all middleware should return a function that takes an `options` object, as in the [Middleware Example](#middleware-example)
* **Return `file`**: all middleware should return the `file` object after processing.

## Globbing examples

Note that the `gitignore` option is already `true` by default, it's just shown here as a placeholder for how options may be defined.

### async

```js
var glob = require('glob-fs')({ gitignore: true });

glob.readdir('**/*.js', function(err, files) {
  console.log(files);
});
```

### promise

```js
var glob = require('glob-fs')({ gitignore: true });

glob.readdirPromise('**/*')
  .then(function (files) {
    console.log(files);
  });
```

### stream

```js
var glob = require('glob-fs')({ gitignore: true });

glob.readdirStream('**/*')
  .on('data', function (file) {
    console.log(file.path);
  })
```

### sync

```js
var glob = require('glob-fs')({ gitignore: true });

var files = glob.readdirSync('**/*.js');
console.log(files);
```

## Events

_(WIP)_

The following events are emitted with all "read" methods:

* `include`: emits a `file` object when it's matched
* `exclude`: emits a `file` object when it's ignored/excluded
* `file`: emits a `file` object when the iterator pushes it into the results array. Only applies to `sync`, `async` and `promise`.
* `dir`: emits a `file` object when the iterator finds a directory
* `end` when the iterator is finished reading
* `error` on errors

### Event examples

**async**

```js
var glob = require('..')({ gitignore: true });

glob.on('dir', function (file) {
  console.log(file);
});

glob.readdir('**/*.js', function (err, files) {
  if (err) return console.error(err);
  console.log(files.length);
});
```

**promise**

```js
var glob = require('glob-fs')({ gitignore: true });

glob.on('include', function (file) {
  console.log('including:', file.path);
});

glob.on('exclude', function (file) {
  console.log('excluding:', file.path);
});

glob.readdirPromise('**/*');
```

**sync**

Also has an example of a custom event, emitted from a middleware:

```js
var glob = require('glob-fs')({ gitignore: true })
  .use(function (file) {
    if (/\.js$/.test(file.path)) {
      // custom event
      this.emit('js', file);
    }
    return file;
  });

glob.on('js', function (file) {
  console.log('js file:', file.path);
});

glob.on('exclude', function (file) {
  console.log('excluded:', i.excludes++);
});

glob.on('include', function (file) {
  console.log('included:', i.includes++)
});

glob.on('end', function () {
  console.log('total files:', this.files.length);
});

glob.readdirSync('**/*.js');
```

**stream**

```js
var glob = require('glob-fs')({ gitignore: true })

glob.readdirStream('**/*')
  .on('data', function (file) {
    console.log(file.path)
  })
  .on('error', console.error)
  .on('end', function () {
    console.log('end');
  });
```

## [node-glob](https://github.com/isaacs/node-glob/)comparison

_(TODO)_

## TODO

* [ ] Multiple pattern support. will need to change pattern handling, middleware handling. this is POC currently
* [ ] Negation patterns (might not do this, since it can be handled in middleware)
* [x] middleware
* [x] middleware handler
* [ ] externalize middleware to modules (started, [prs welcome!](#contributing))
* [x] events
* [x] unit tests (need to be moved)
* [x] sync iterator
* [x] async iterator
* [x] stream iterator
* [x] promise iterator
* [x] glob.readdir (async)
* [x] glob.readdirSync
* [x] glob.readdirStream
* [x] glob.readdirPromise
* [ ] clean up `./lib`

## Related projects

* [braces](https://github.com/jonschlinkert/braces): Fastest brace expansion for node.js, with the most complete support for the Bash 4.3 braces… [more](https://github.com/jonschlinkert/braces)
* [fill-range](https://github.com/jonschlinkert/fill-range): Fill in a range of numbers or letters, optionally passing an increment or multiplier to… [more](https://github.com/jonschlinkert/fill-range)
* [is-glob](https://github.com/jonschlinkert/is-glob): Returns `true` if the given string looks like a glob pattern.
* [micromatch](https://github.com/jonschlinkert/micromatch): Glob matching for javascript/node.js. A drop-in replacement and faster alternative to minimatch and multimatch. Just… [more](https://github.com/jonschlinkert/micromatch)

## Running tests

Install dev dependencies:

```sh
$ npm i -d && npm test
```

## Contributing

Pull requests and stars are always welcome. For bugs and feature requests, [please create an issue](https://github.com/jonschlinkert/glob-fs/issues/new).

## Author

**Jon Schlinkert**

+ [github/jonschlinkert](https://github.com/jonschlinkert)
+ [twitter/jonschlinkert](http://twitter.com/jonschlinkert)

## License

Copyright © 2015 Jon Schlinkert
Released under the MIT license.

***

_This file was generated by [verb-cli](https://github.com/assemble/verb-cli) on July 07, 2015._