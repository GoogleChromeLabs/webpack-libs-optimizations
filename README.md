# Optimize your libraries with webpack

Using a library in your webpack project? Use these tips to make your bundle smaller!

Want to add a tip? See [the contribution guide](/CONTRIBUTING.md) and make a pull request!

Contents:

* [`babel-polyfill`](#babel-polyfill)
* [`core-js`](#core-js)
* [`lodash`](#lodash)
* [`lodash-es`](#lodash-es)
* [`moment`](#moment)
* [`react`](#react)
* [`styled-components`](#styled-components)
* [`whatwg-fetch`](#whatwg-fetch)

## babel-polyfill

`babel-polyfill` is a Babel’s package that loads `core-js` and a custom regenerator runtime. [Babel docs](https://babeljs.io/docs/usage/polyfill/) · [npm package](https://www.npmjs.com/package/babel-polyfill)

For the list of optimizations, see [the `core-js` section](#core-js).

## core-js

`core-js` is a set of polyfills for ES5 and ES2015+. [npm package](https://www.npmjs.com/package/core-js)

### Remove unnecessary polyfills with `babel-preset-env`

> ✅ Safe to use by default / [How to enable](https://github.com/lodash/babel-plugin-lodash) / Added by [@iamakulov](https://twitter.com/iamakulov)

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

## lodash

Lodash is an utility library. [npm package](https://www.npmjs.com/package/lodash)

### Enable `babel-plugin-lodash`

> ✅ Safe to use by default / [How to enable](https://github.com/lodash/babel-plugin-lodash) / Added by [@iamakulov](https://twitter.com/iamakulov)

[`babel-plugin-lodash`](https://github.com/lodash/babel-plugin-lodash) replaces full imports of Lodash with imports of specific Lodash functions:

```
import _ from 'lodash'
_.map([1, 2, 3], i => i + 1)
```

↓

```
import _map from 'lodash/map'
_map([1, 2, 3], i => i + 1)
```

Note: the plugin doesn’t work with chain sequences – i.e. code like

```
_([1, 2, 3]).map(i => i + 1).value()
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

For the list of optimizations, see [the `lodash` section](#lodash).

## moment

Moment.js is a library for working with dates. [npm package](https://www.npmjs.com/package/moment)

### Remove unused locales with `moment-locales-webpack-plugin`

> ⚠ Use with caution / [How to enable](https://github.com/iamakulov/moment-locales-webpack-plugin) / Added by [@iamakulov](https://twitter.com/iamakulov)

By default, Moment.js ships with 160+ minified KBs of localization files. If you app is only available in a few languages, you won’t need all these files. Use [`moment-locales-webpack-plugin`](https://github.com/iamakulov/moment-locales-webpack-plugin) to remove the unused ones.

**Use the plugin with cautio.** The default settings remove all locales; this might break your app if you use some of them.

## react

React is a library for building user interfaces. [npm package](https://www.npmjs.com/package/react)

### Remove `propTypes` declarations in production

> ✅ Safe to use by default / [How to enable](https://www.npmjs.com/package/babel-plugin-transform-react-remove-prop-types) / Added by [@iamakulov](https://twitter.com/iamakulov)

React doesn’t perform `propTypes` checks in production, but the `propTypes` declarations still occupy a part of the bundle. Use [`babel-plugin-transform-react-remove-prop-types`](https://www.npmjs.com/package/babel-plugin-transform-react-remove-prop-types) to remove them from during building.

### Replace with Preact

> ⚠ Use with caution / [How to migrate](https://preactjs.com/guide/switching-to-preact) / Added by [@iamakulov](https://twitter.com/iamakulov)

[Preact](https://github.com/developit/preact) is a smaller React alternative with a similar API. Switching to it saves you 250 minified KBs (tested with `preact@8.2.7` + `preact-compat@3.17.0` vs. `react@16.2.0` + `react-dom@16.2.0`).

**Migrate to Preact with caution.** Preact is not 100% compatible with React – i.e. it doesn’t support synthetic events and [some React 16 features](https://github.com/developit/preact-compat/issues/432). However, many projects still can be migrated without any codebase changes. See [the migration guide](https://preactjs.com/guide/switching-to-preact).

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
