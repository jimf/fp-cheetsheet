# Promises as Futures

The `Future` (sometimes also referred to as `Task`) structure is a monad for
dealing with asynchronous events in a pure and principled way. In order to
retain purity, rather than perform async operations directly, a Future instead
wraps a function that performs the async operation and exposes a method for
running that function on demand. This allows for the side effect to be pushed
toward the edges of the given application. Below we'll look at some typical
Promise-based solutions to async, paired with reimplementations that use
Futures.

## Contents

 * [Getting Started](#getting-started)
 * [Construction](#construction)
 * [Operation Chaining](#operation-chaining)
 * [Parallel Execution](#parallel-execution)
 * [Resources](#resources)

## Getting Started

Examples assume Node.js 4+.

    $ npm install ramda data.task

Initial requires:

```js
var fs = require('fs');
var Future = require('data.task');
var R = require('ramda');
```

## Construction

#### Promise

```js
// Function that makes an async request and returns a Promise
function readdir(path) {
    return new Promise(function(resolve, reject) {
        fs.readdir(path, function(err, files) {
            if (err) {
                reject(err);
            } else {
                resolve(files);
            }
        });
    });
}

function readfile(path) {
    return new Promise(function(resolve, reject) {
        fs.readFile(path, 'utf8', function(err, files) {
            if (err) {
                reject(err);
            } else {
                resolve(files);
            }
        });
    });
}

// Run readdir:
readdir('.');

// Create a resolved promise:
Promise.resolve(value);

// Create a rejected promise:
Promise.reject(value);
```

#### Future

```js
// Function that returns a Future that can be forked to make an async request
function readdir(path) {
    // NOTE: reject/resolve order is flipped!
    return new Future(function(reject, resolve) {
        fs.readdir(path, function(err, files) {
            if (err) {
                reject(err);
            } else {
                resolve(files);
            }
        });
    });
}

function readfile(path) {
    // NOTE: reject/resolve order is flipped!
    return new Future(function(reject, resolve) {
        fs.readFile(path, 'utf8', function(err, files) {
            if (err) {
                reject(err);
            } else {
                resolve(files);
            }
        });
    });
}

// Run readdir:
readdir('.').fork(console.warn, console.log);

// Create a resolved Future:
Future.of(value);

// Create a rejected Future:
Future.rejected(value);
```

## Operation Chaining

#### Promise

```js
function listJsFiles(path) {
    return readdir(path).then(R.filter(R.test(/\.js$/)));
}

listJsFiles('.').then(console.log).catch(console.warn);

function readFirstJsFile(path) {
    return listJsFiles(path).then(R.head).then(readfile)
}

readFirstJsFile('.').then(console.log).catch(console.warn);
```

#### Future

```js
const listJsFiles = R.compose(R.map(R.filter(R.test(/\.js$/))),
                              readdir);

listJsFiles('.').fork(console.warn, console.log);

const readFirstJsFile = R.compose(R.chain(readfile),
                                  R.map(R.head),
                                  listJsFiles);

readFirstJsFile('.').fork(console.warn, console.log);
```

Key points:

- `map` to run functions over `Future` values
- `chain` when working with Futures within Futures (`chain` is `map` + "flatten")

## Parallel Execution

#### Promise

```js
function readCurrentAndParentDirs() {
    return Promise.all([readdir('.'), readdir('..')])
}

// Read current and parent directories and log once both requests are complete:
readCurrentAndParentDirs().then(console.log).catch(console.warn);
```

#### Future

```js
const readCurrentAndParentDirs = () =>
    R.lift(current => parent => [current, parent])(readdir('.'), readdir('..'));

// Read current and parent directories and log once both requests are complete:
readCurrentAndParentDirs().fork(console.warn, console.log);
```

Key points:

- the function given to `R.lift` MUST:
  - be curried
  - have an arity equal to the number of actions being taken (in this case, 2)

## Resources

- [Monad a day 2: Future](https://vimeo.com/106008027)
- [Mostly Adequate Guide, Ch. 8.6: Asynchronous Tasks](https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch8.html)
