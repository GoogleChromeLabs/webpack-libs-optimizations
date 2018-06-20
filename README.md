# Optimize your libraries with webpack

Using a library in your webpack project? Use these tips to make your bundle smaller!

Want to add a tip? See [the contribution guide](/CONTRIBUTING.md) and make a pull request!

<img src="https://i.imgur.com/tjFWoUj.png" width="600" />

Contents:

* [`async`](#async)
* [`async-es`](#async-es)
* [`babel-polyfill`](#babel-polyfill)
* [`core-js`](#core-js)
* [`date-fns`](#date-fns)
* [`lodash`](#lodash)
* [`lodash-es`](#lodash-es)
* [`moment`](#moment)
* [`react`](#react)
* [`reactstrap`](#reactstrap)
* [`react-bootstrap`](#react-bootstrap)
* [`react-router`](#react-router)
* [`styled-components`](#styled-components)
* [`whatwg-fetch`](#whatwg-fetch)
* [Solutions that work with multiple libraries](#solutions-that-work-with-multiple-libraries)

## async

`async` is a collection of utilities for working with async functions. [npm package](https://www.npmjs.com/package/async)

Generally, you should use [the `async-es` package ⤵](#async-es) instead. It ships with ES modules and is more optimized for bundling with webpack.

Still, even if prefer to use `async`, for the list of optimizations, see [the `async-es` section ⤵](#async-es).

## async-es

`async-es` is a collection of utilities for working with async functions. It’s the same package as [`async` ⤴](#async), but it’s more optimized for bundling with webpack. [npm package](https://www.npmjs.com/package/async-es)

### Remove unused methods with `babel-plugin-lodash`

> ✅ Safe to use by default / How to enable is ↓ / Added by [@iamakulov](https://twitter.com/iamakulov)

If you use `async-es` as a single import, you’re bundling the whole library into the application – even if you only use a couple of its methods:

```js
// You only use `async.map`, but the whole library gets bundled
import async from 'async-es';

async.map(['file1', 'file2', 'file3'], fs.stat, function(err, results) {
  console.log(results);
});
```

Use [`babel-plugin-lodash`](https://github.com/lodash/babel-plugin-lodash) to pick only those `async-es` methods you need:

```js
// Before Babel is applied
import async from 'async-es';

async.map(['file1', 'file2', 'file3'], fs.stat, function(err, results) {
  console.log(results);
});

↓

// After Babel is applied
import _map from 'async-es/map';

_map(['file1', 'file2', 'file3'], fs.stat, function(err, results) {
  console.log(results);
});
```

Enable this plugin as follows:

```json
// .babelrc
{
  "plugins": [["lodash", { "id": ["async-es"] }]],
}
```

## babel-polyfill

`babel-polyfill` is a Babel’s package that loads `core-js` and a custom regenerator runtime. [Babel docs](https://babeljs.io/docs/usage/polyfill/) · [npm package](https://www.npmjs.com/package/babel-polyfill)

For the list of optimizations, see [the `core-js` section ⤵](#core-js).

## core-js

`core-js` is a set of polyfills for ES5 and ES2015+. [npm package](https://www.npmjs.com/package/core-js)

### Remove unnecessary polyfills with `babel-preset-env`

> ✅ Safe to use by default / [How to enable](https://babeljs.io/docs/plugins/preset-env/#usebuiltins) / Added by [@iamakulov](https://twitter.com/iamakulov)

If you compile your code with Babel and `babel-preset-env`, add [the `useBuiltIns: true` option](https://babeljs.io/docs/plugins/preset-env/#usebuiltins). This option configures Babel to only include polyfills that are necessary for target browsers. I.e., if you target your app to support Internet Explorer 11:

```json
// .babelrc
{
  "presets": [
    [
      "env",
      {
        "targets": {
          "browsers": ["last 2 versions", "ie >= 11"]
        }
      }
    ]
  ]
}
```

enabling `useBuiltIns: true` will remove polyfills for all features that Internet Explorer 11 already supports (such as `Object.create`, `Object.keys` and so on).

### Ship non-transpiled code to modern browsers

> ✅ Safe to use by default / [How to enable](https://philipwalton.com/articles/deploying-es2015-code-in-production-today/) / Added by [@iamakulov](https://twitter.com/iamakulov)

All browsers that support `<script type="module">` also support modern JS features like `async`/`await`, arrow functions and classes. Use this feature to build two versions of the bundle and make modern browsers load only the modern code. For the guide, see [the Philip Walton’s article](https://philipwalton.com/articles/deploying-es2015-code-in-production-today/).

## date-fns

date-fns is a date utility library. [npm package](https://www.npmjs.com/package/date-fns)

### Enable `babel-plugin-date-fns`

> ✅ Safe to use by default / [How to enable](https://github.com/date-fns/babel-plugin-date-fns) / Added by [@chentsulin](https://twitter.com/chentsulin)

[`babel-plugin-date-fns`](https://github.com/date-fns/babel-plugin-date-fns) replaces full imports of date-fns with imports of specific date-fns functions:

```js
import { format } from 'date-fns';
format(new Date(2014, 1, 11), 'MM/DD/YYYY');
```

↓

```js
import _format from 'date-fns/format';
_format(new Date(2014, 1, 11), 'MM/DD/YYYY');
```

## lodash

Lodash is an utility library. [npm package](https://www.npmjs.com/package/lodash)

### Enable `babel-plugin-lodash`

> ✅ Safe to use by default / [How to enable](https://github.com/lodash/babel-plugin-lodash) / Added by [@iamakulov](https://twitter.com/iamakulov)

[`babel-plugin-lodash`](https://github.com/lodash/babel-plugin-lodash) replaces full imports of Lodash with imports of specific Lodash functions:

```js
import _ from 'lodash';
_.map([1, 2, 3], i => i + 1);
```

↓

```js
import _map from 'lodash/map';
_map([1, 2, 3], i => i + 1);
```

Note: the plugin doesn’t work with chain sequences – i.e. code like

```js
_([1, 2, 3]).map(i => i + 1).value();
```

won’t be optimized.

### Alias `lodash-es` to `lodash`

> ✅ Safe to use by default / How to enable is ↓ / Added by [@7rulnik](https://twitter.com/7rulnik)

Some of your dependencies might use [the `lodash-es` package](https://www.npmjs.com/package/lodash-es) instead of `lodash`. If that’s the case, Lodash will be bundled twice.

To avoid this, alias the `lodash-es` package to `lodash`:

```js
// webpack.config.js
module.exports = {
  resolve: {
    alias: {
      'lodash-es': 'lodash',
    },
  },
};
```

### Enable `lodash-webpack-plugin`

> ⚠ Use with caution / [How to enable](https://github.com/lodash/lodash-webpack-plugin) / Added by [@iamakulov](https://twitter.com/iamakulov)

[`lodash-webpack-plugin`](https://github.com/lodash/lodash-webpack-plugin) strips parts of Lodash functionality that you don’t need. For example, if you use `_.get()` but don’t need deep path support, this plugin can remove it. Add it to your webpack config to make the bundle smaller.

**Use the plugin with caution.** The default settings remove a lot of features, and your app might use some of them.

## lodash-es

`lodash-es` is Lodash with ES imports and exports. [npm package](https://www.npmjs.com/package/lodash-es)

For the list of optimizations, see [the `lodash` section ⤴](#lodash).

## moment

Moment.js is a library for working with dates. [npm package](https://www.npmjs.com/package/moment)

### Remove unused locales with `moment-locales-webpack-plugin`

> ⚠ Use with caution / [How to enable](https://github.com/iamakulov/moment-locales-webpack-plugin) / Added by [@iamakulov](https://twitter.com/iamakulov)

By default, Moment.js ships with 160+ minified KBs of localization files. If you app is only available in a few languages, you won’t need all these files. Use [`moment-locales-webpack-plugin`](https://github.com/iamakulov/moment-locales-webpack-plugin) to remove the unused ones.

**Use the plugin with caution.** The default settings remove all locales; this might break your app if you use some of them.

## react

React is a library for building user interfaces. [npm package](https://www.npmjs.com/package/react)

### Remove `propTypes` declarations in production

> ✅ Safe to use by default / [How to enable](https://www.npmjs.com/package/babel-plugin-transform-react-remove-prop-types) / Added by [@iamakulov](https://twitter.com/iamakulov)

React doesn’t perform `propTypes` checks in production, but the `propTypes` declarations still occupy a part of the bundle. Use [`babel-plugin-transform-react-remove-prop-types`](https://www.npmjs.com/package/babel-plugin-transform-react-remove-prop-types) to remove them from during building.

### Migrate to an alternative React-like Library

> ⚠ Use with caution / Added by [@iamakulov](https://twitter.com/iamakulov) & [@kurtextrem](https://twitter.com/kurtextrem)

There are alternatives to React, that share the same principle and similar APIs, but with a much tinier footprint or a lot higher performance. Mind you: Those don't support the anticipated "Suspension" and "Fiber" features from React.

You can find a benchmark over here (use with caution and ymmv): https://rawgit.com/krausest/js-framework-benchmark/master/webdriver-ts-results/table.html

- [Preact](https://github.com/developit/preact) | smallest React alternative (switching to it saves you 250 minified KBs (tested with `preact@8.2.7` + `preact-compat@3.17.0` vs. `react@16.2.0` + `react-dom@16.2.0`)) | No synthetic events | IE8 supported with polyfills

- [Inferno](https://github.com/infernojs/inferno) | smaller than React, larger than Preact (~8kb vs 3kb gziped) | Higher performance than React,  highest performance among all React alternatives and offers [manual optimization possibilitys](https://infernojs.org/docs/guides/optimizations) | Partial synthetic events | IE8 unsupported natively

- [Nerv](https://github.com/NervJS/nerv) | smaller than React, larger than Preact and Inferno (9kb gziped) | The goal of Nerv is to have 100% the same API (without Fiber and Suspense), see [this](https://github.com/NervJS/nerv/issues/10) issue for details | IE8 supported

**Migrate to alternatives with caution.** Some of the alternatives don’t have synthetic events or are lacking some React 16 features ([Preact](https://github.com/developit/preact-compat/issues/432), [Inferno](https://github.com/infernojs/inferno/issues/501), [Nerv](https://github.com/NervJS/nerv/issues/5)). However, many projects still can be migrated without any codebase changes. See the migration guides: [Preact](https://preactjs.com/guide/switching-to-preact), [Inferno](https://infernojs.org/docs/guides/switching-to-inferno), [Nerv](https://github.com/NervJS/nerv#switching-to-nerv-from-react).

## reactstrap

Reactstrap is a Bootstrap 4 library for React. [npm package](https://www.npmjs.com/package/reactstrap)

### Remove unused modules with `babel-plugin-transform-imports`

> ✅ Safe to use by default / How to enable is ↓ / Added by [@kurtextrem](https://twitter.com/kurtextrem)

When you import a module from Reactstrap:

```js
import { Alert } from 'reactstrap';
```

other Reactstrap modules also get bundled into the app and make it larger.

Use [`babel-plugin-transform-imports`](https://www.npmjs.com/package/babel-plugin-transform-imports) to strip unused modules:

```json
// .babelrc
{
  "plugins": [
    ["transform-imports", {
      "reactstrap": {
        "transform": "reactstrap/lib/${member}",
        "preventFullImport": true
      }
    }]
  ]
}
```

To see how it works, check [the `babel-plugin-transform-imports` section ⤵️](#babel-plugin-transform-imports).

## react-bootstrap

`react-bootstrap` is a Bootstrap 3 library for React. [npm package](https://www.npmjs.com/package/react-bootstrap)

### Remove unused modules with `babel-plugin-transform-imports`

> ✅ Safe to use by default / How to enable is ↓ / Added by [@kurtextrem](https://twitter.com/kurtextrem)

When you import a module from `react-bootstrap`:

```js
import { Alert } from 'react-bootstrap';
```

other `react-bootstrap` modules also get bundled into the app and make it larger.

Use [`babel-plugin-transform-imports`](https://www.npmjs.com/package/babel-plugin-transform-imports) to strip unused modules:

```json
// .babelrc
{
  "plugins": [
    ["transform-imports", {
      "react-bootstrap": {
        "transform": "react-bootstrap/es/${member}",
        "preventFullImport": true
      }
    }]
  ]
}
```

To see how it works, check [the `babel-plugin-transform-imports` section ⤵️](#babel-plugin-transform-imports).

## react-router

React Router is a popular router solution for React. [npm package](https://www.npmjs.com/package/react-router)

### Remove unused modules with `babel-plugin-transform-imports`

> ✅ Safe to use by default / How to enable is ↓ / Added by [@kurtextrem](https://twitter.com/kurtextrem)

When you import a module from React Router:

```js
import { withRouter } from 'react-router';
```

other React Router modules also get bundled into the app and make it larger.

Use [`babel-plugin-transform-imports`](https://www.npmjs.com/package/babel-plugin-transform-imports) to strip unused modules:

```json
// .babelrc
{
  "plugins": [
    ["transform-imports", {
      "react-router": {
        "transform": "react-router/${member}",
        "preventFullImport": true
      }
    }]
  ]
}
```

(This was tested with React Router v4.)

To see how it works, check [the `babel-plugin-transform-imports` section ⤵️](#babel-plugin-transform-imports).

## styled-components

`styled-components` is a CSS-in-JS library. [npm package](https://www.npmjs.com/package/styled-components)

### Minify the code with `babel-plugin-styled-components`

> ✅ Safe to use by default / [How to enable](https://github.com/styled-components/babel-plugin-styled-components) / Added by [@iamakulov](https://twitter.com/iamakulov)

There’s [`babel-plugin-styled-components`](https://github.com/styled-components/babel-plugin-styled-components) that minifies the CSS code you write with `styled-components`. See [the minification docs](https://www.styled-components.com/docs/tooling#minification).

## whatwg-fetch

`whatwg-fetch` is a complete `window.fetch()` polyfill. [npm package](https://www.npmjs.com/package/whatwg-fetch)

### Replace with `unfetch`

> ⚠ Use with caution / How to migrate is ↓ / Added by [@iamakulov](https://twitter.com/iamakulov)

[`unfetch`](https://github.com/developit/unfetch) is a 500 bytes polyfill for `window.fetch()`. Unlike `whatwg-fetch`, it doesn’t support the full `window.fetch()` API, but instead focuses on polyfilling the most used parts.

**Migrate to `unfetch` with caution.** While it supports the most popular API parts, your app might break if it relies on something less common.

## Solutions that work with multiple libraries

Of course, there are also optimization tips for other libraries too. You can use them with common sense to get smaller or more performant bundles.

### `babel-plugin-transform-imports`

> ✅ Safe to use by default / [How to enable](https://www.npmjs.com/package/babel-plugin-transform-imports) / Added by [@kurtextrem](https://twitter.com/kurtextrem) / More Insight about this on [Twitter](https://twitter.com/iamakulov/status/962991382213398529)

This handy babel plugin will transform your imports to only import specific components, which ensures not the whole library gets included (if tree-shaking is ineffective for the specific library).
```js
// Before
import { Grid, Row, Col } from 'react-bootstrap';
// After
import Grid from 'react-bootstrap/lib/Grid';
import Row from 'react-bootstrap/lib/Row';
import Col from 'react-bootstrap/lib/Col';
```

# License

Copyright 2018 Google Inc. All Rights Reserved. Licensed under [the Apache License, Version 2.0](/LICENSE).
