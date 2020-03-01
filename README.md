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
* [`handlebars`](#handlebars)
* [`lodash`](#lodash)
* [`lodash-es`](#lodash-es)
* [`moment`](#moment)
* [`moment-timezone`](#moment-timezone)
* [`react`](#react)
* [`ractive`](#ractive)
* [`reactstrap`](#reactstrap)
* [`react-bootstrap`](#react-bootstrap)
* [`react-router`](#react-router)
* [`styled-components`](#styled-components)
* [`whatwg-fetch`](#whatwg-fetch)
* [Solutions that work with multiple libraries](#solutions-that-work-with-multiple-libraries)
    * [`babel-plugin-transform-imports`](#babel-plugin-transform-imports)
    * [Templating languages: bundle just the runtime](#templating-languages-bundle-just-the-runtime)

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

## handlebars

Handlebars is a templating library. [npm package](https://www.npmjs.com/package/handlebars)

### Switch to build-time compiling

> ⚠ Use with caution / How to enable is ↓ / Added by [@danburzo](https://github.com/danburzo)

If you use `Handlebars.compile()` to compile templates in your app, switch to [`handlebars-loader`](https://github.com/pcardune/handlebars-loader). This way, you’ll compile templates during a webpack build – and won’t need to bundle the template-parsing part of the library.

Here’s how to avoid bundling the template-parsing part of Handlebars:

* Switch to [`handlebars-loader`](https://github.com/pcardune/handlebars-loader), if you haven’t yet
* And either:
   * replace all `import Handlebars from 'handlebars'` with `import Handlebars from 'handlebars/runtime'`
   * or alias the module using `resolve.alias`:
   
      ```js
      // webpack.config.js
      {
        // ...
        resolve: {
          alias: {
            // Tip: `$` in the end of `handlebars$` means “exact match”: https://webpack.js.org/configuration/resolve/#resolvealias
            // This’d disable aliasing – and prevent breaking the code – for imports like `handlebars/something.js`
            handlebars$: path.resolve(__dirname, 'node_modules/handlebars/runtime.js')
          }
        }
        // ...
      }
      ```

**Use this optimization with caution.** Make sure your code does not use `Handlebars.compile()` anywhere, or your app will break.

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

By default, Moment.js ships with 160+ minified KBs of localization files. If your app is only available in a few languages, you won’t need all these files. Use [`moment-locales-webpack-plugin`](https://github.com/iamakulov/moment-locales-webpack-plugin) to remove the unused ones.

**Use the plugin with caution.** The default settings remove all locales; this might break your app if you use some of them.

## moment-timezone

Moment Timezone is a plugin for Moment.js with full support for time zone calculations. [npm package](https://www.npmjs.com/package/moment-timezone)

### Remove unused data with `moment-timezone-data-webpack-plugin`

> ⚠ Use with caution / [How to enable](https://github.com/gilmoreorless/moment-timezone-data-webpack-plugin) / Added by [@iamnotyourbroom](https://twitter.com/iamnotyourbroom)

By default, Moment Timezone includes as much time zone data as possible via a 900+ KB JSON file. In some cases this data includes dates in the 19th century. If your app can work with a smaller range of dates, or only needs specific time zones, most of that data is redundant. Use [`moment-timezone-data-webpack-plugin`](https://github.com/gilmoreorless/moment-timezone-data-webpack-plugin) to remove the unused data.

**Use the plugin with caution.** Removing too much time zone data can cause subtle date calculation bugs. Make sure your app still has all the data it needs to function correctly.

## ractive

Ractive is a UI templating library. [npm package](https://www.npmjs.com/package/ractive)

### Switch to build-time compiling

> ⚠ Use with caution / How to enable is ↓ / Added by [@danburzo](https://github.com/danburzo)

If you’re compiling your Ractive templates on the go (e.g., by passing strings to `Ractive({ template })`, switch to [`ractive-loader`](https://www.npmjs.com/package/ractive-loader). This way, you’ll compile templates during a webpack build – and won’t need to bundle the template-parsing part of the library.

Here’s how to avoid bundling the template-parsing part of Ractive:

* Switch to [`ractive-loader`](https://www.npmjs.com/package/ractive-loader), if you haven’t yet
* And alias the `ractive` module to the Ractive runtime using `resolve.alias`:
   
   ```js
   // webpack.config.js
   {
     // ...
     resolve: {
       alias: {
         // Tip: `$` in the end of `ractive$` means “exact match”: https://webpack.js.org/configuration/resolve/#resolvealias
         // This’d disable aliasing – and prevent breaking the code – for imports like `ractive/something.js`
         ractive$: path.resolve(__dirname, 'node_modules/ractive/runtime.min.js')
       }
     }
     // ...
   }
   ```
   
**Use this optimization with caution.** Make sure your code does not compile any templates on the fly, or your app will break. Compiling templates on the fly happens whenever you pass a string to `Ractive({ template: ... })` or `Ractive.parse()`.

## react

React is a library for building user interfaces. [npm package](https://www.npmjs.com/package/react)

### Remove `propTypes` declarations in production

> ✅ Safe to use by default / [How to enable](https://www.npmjs.com/package/babel-plugin-transform-react-remove-prop-types) / Added by [@iamakulov](https://twitter.com/iamakulov)

React doesn’t perform `propTypes` checks in production, but the `propTypes` declarations still occupy a part of the bundle. Use [`babel-plugin-transform-react-remove-prop-types`](https://www.npmjs.com/package/babel-plugin-transform-react-remove-prop-types) to remove them from during building.

### Migrate to an alternative React-like Library

> ⚠ Use with caution / Added by [@iamakulov](https://twitter.com/iamakulov) & [@kurtextrem](https://twitter.com/kurtextrem)

There are alternatives to React with a similar API that have a smaller size or a higher performance, but lack some features (e.g., fragments, portals, or synthetic events).

- [Preact](https://github.com/developit/preact) | The smallest React alternative (`preact@8.3.1` + `preact-compat@3.18.3` is 7.6 kB gzipped; `react@16.4.0` + `react-dom@16.4.0` is 31.4 kB gzipped) | No synthetic events | IE8 supported with polyfills

- [Nerv](https://github.com/NervJS/nerv) | Smaller than React, larger than Preact (`nervjs@1.3.3` is 9.8 kB gzipped, compat is not needed; `react@16.4.0` + `react-dom@16.4.0` is 31.4 kB gzipped) | The goal of Nerv is to have 100% the same API (without Fiber and Suspense), see [NervJS/nerv#10](https://github.com/NervJS/nerv/issues/10#issuecomment-356913486) for details | IE8 supported

- [Inferno](https://github.com/infernojs/inferno) | Smaller than React, larger than Preact and Nerv (`inferno@5.4.2` + `inferno-compat@5.4.2` is 11.3 kB gzipped; `react@16.4.0` + `react-dom@16.4.0` is 31.4 kB gzipped) | Higher runtime performance than React, the highest performance among all React alternatives, [manual optimization possibilities offered](https://infernojs.org/docs/guides/optimizations) | Partial synthetic events | IE8 unsupported natively

**Migrate to alternatives with caution.** Some of the alternatives don’t have synthetic events or are lacking some React 16 features ([Preact issue](https://github.com/developit/preact-compat/issues/432), [Inferno issue](https://github.com/infernojs/inferno/issues/501), [Nerv issue](https://github.com/NervJS/nerv/issues/5)). However, many projects still can be migrated without any codebase changes. See the migration guides: [Preact](https://preactjs.com/guide/switching-to-preact), [Inferno](https://infernojs.org/docs/guides/switching-to-inferno), [Nerv](https://github.com/NervJS/nerv#switching-to-nerv-from-react).

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
