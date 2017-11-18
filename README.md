# @rjh/export-dir

Declarative `index.js` builder for exporting files in the same directory.

This tiny package removes the maintenance of updating `index.js` files that simply `require` and export all of the files in the same directory to allow for a deconstuctable interface of the folder.



## Installation

```
yarn add @rjh/export-dir
```
```
npm install --save @rjh/export-dir
```



## Usage

**Basic Usage**

```js
module.exports = require('@rjh/export-dir')(null, __dirname)
```

**With Default `transform`**

```js
const camelCase = require('lodash.camelcase')
module.exports = require('@rjh/export-dir')(camelcase, __dirname)
```

**With Custom `transform`**

```js
const { compose } = require('ramda')
const camelCase = require('lodash.camelcase')
const upperFirst = require('lodash.upperfirst')
module.exports =
  require('@rjh/export-dir')(
    compose(upperFirst, camelCase),
    __dirname,
  )
```



## Problem / Solution

Imagine we have the following project.

```
/
  app.js
  lib/
    foo.js
    foo.test.js
    bar.js
    bar.test.js
    baz.js
    baz.test.js
    index.js
```

We want to make sure we're keeping our functions isolated, testable, and generic, so we created a `lib` folder to keep them. This also has the advantage of keeping our `app.js` clean of helper methods and just focusing on what it, specifically, wants to do.

We want to avoid doing this:

```js
// app.js
const foo = require('./lib/foo')
const bar = require('./lib/bar')
const baz = require('./lib/baz')
// ...
```

So we create `lib/index.js` that rexports everything...

```js
// lib/index.js
module.exports = {
  foo: require('./foo'),
  bar: require('./bar'),
  baz: require('./baz'),
}
```

Great. New we can change our `app.js` file to deconstruct the methods from the folder require.

```js
// app.js
const { foo, bar, baz } = require('./lib');
// ...
```

We're done, except that this is a pain to maintain. Every time we add a new method, we have to go update our index file. If we forget, we'll have a runtime error when trying to deconstruct our new method in app. This package aims to solve the problem by allowing you to setup rules for how the index file should be built, and then not having to worry about it anymore.

Here's what our `lib/index.js` and `app.js` looks like when using `@rjh/export-dir`:

```js
// lib/index.js
const exportDir = require('@rjh/export-dir')
module.exports = exportDir(null, __dirname)

// app.js
const { foo, bar, baz } = require('./lib');
```



## Docs

### Signature

The function is autocurried, so you can call it however you would like, with a caveat being that you must call it will all parameters.

`exportDir :: (String -> String) => String<Dir> -> Object`
`exportDir :: (transformation, path) => ({ })`

### Behavior

Something more formal will come, but for now, here's some notes about the behavior.

- All `.js` and `.json` files will be exported.
- `.tests.js` files will be ignored.
- `index.js` will be ignored.
- `lodash.camelcase` is the default transformation.



## Q & A

**Who cares about CommonJS modules anymore?**
> Though ES modules in Node.js is right around the corner, this is still a pattern that will likely be around in legacy code for some time.

**Why does this need to exist when there's others that have implemented the same thing, and with more features?**
> I didn't find one that had everything I wanted and I didn't want to work in their respective code bases. I just made this for me. I'm surprised you're even reading this.
