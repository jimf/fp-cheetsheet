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
 * [Leveraging Natural Transformations](#leveraging-natural-transformations)
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
// Read the given directory and return a Promise.
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

// Read the given file and return a Promise.
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
// Return a future that will read the given directory when forked.
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

// Return a future that will read the given file when forked.
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

Key points:

- very similar to the `Promise` API, but with `resolve`/`reject` flipped

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
- `chain` when working with Futures within Futures to avoid nesting
  - `chain` is `map` + "flatten"
  - `Future(Future(value))` becomes `Future(value)`

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

// Alternative method:
const readCurrentAndParentDirs2 = () =>
    R.traverse(Future.of, readdir, ['.', '..']);

// Return a list of Future results:
readCurrentAndParentDirs2().fork(console.warn, console.log);
```

Key points:

- the function given to `R.lift` MUST:
  - be curried
  - have an arity equal to the number of actions being taken (in this case, 2)

## Leveraging Natural Transformations

When combining Futures with other monads like `Maybe`, `Either` and the like,
it can be useful to leverage [natural transformations](README.md#natural-transformation)
to immediately bail out from a Future sequence.

For example, lets say we are fetching a result set, taking a piece out of the
result (like an id), and then making a subsequent request. However, to
abstract out the idea of failure, we've bottled up the extraction in an
`Either`. In order to *not* make the second request, we could use a natural
transformation to convert our `Either` directly into a `Future`, and if the
`Either` is a `Left`, have the `Future` be rejected:

```js
const
    R = require('ramda'),
    Either = require('data.either'),
    Future = require('data.task'),
    request = require('request');

const Left = Either.Left;
const Right = Either.Right;

// :: -> Future String (Array User)
const getAllUsers = () => new Future((reject, resolve) => {
    request({
        url: 'http://jsonplaceholder.typicode.com/users',
        json: true
    }, (err, response, body) => (
        err || response.statusCode !== 200
            ? reject('Failed to fetch all users')
            : resolve(body)
    ))
});

// :: Int -> Future String (Array User)
const getUser = id => new Future((reject, resolve) => {
    request({
        url: `http://jsonplaceholder.typicode.com/users/${id}`,
        json: true
    }, (err, response, body) => (
        err || response.statusCode !== 200
            ? reject(`Failed to fetch user ${id}`)
            : resolve(body)
    ))
});

// :: Array User -> Either String User
const firstUser = users => users.length > 0 ? Right(users[0]) : Left('No users found.')

// Natural transformation from Either to Future. If the Either is a Left, the Future
// will be rejected. Otherwise it will be resolved.
//
// :: Either a b -> Future a b
const eitherToFuture = either => either.cata({
    Left: Future.rejected,
    Right: Future.of
});

// Get all users
// -> then pluck out the first user
// -> then grab that user's id property
// -> then fetch the record for that id
getAllUsers()
    .chain(R.compose(eitherToFuture, R.map(R.prop('id')), firstUser))
    .chain(getUser)
    .fork(console.warn, console.log);
```

## Resources

### Information

- [Monad a day 2: Future](https://vimeo.com/106008027)
- [Mostly Adequate Guide, Ch. 8.6: Asynchronous Tasks](https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch8.html)

### Node Modules

- [data.task](https://www.npmjs.com/package/data.task)
- [ramda-fantasy](https://www.npmjs.com/package/ramda-fantasy)
- [fantasy-promises](https://github.com/fantasyland/fantasy-promises)
- [futurize](https://www.npmjs.com/package/futurize)
- [redux-future](https://github.com/stoeffel/redux-future)
