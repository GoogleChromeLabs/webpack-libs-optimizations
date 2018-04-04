Thanks for contributing! To add a tip, follow these three easy steps.

# 1. Choose the proper kind of the tip

There’re two kinds of tips:

* “✅ Safe to use by default” one and
* “⚠ Use with caution” one

The first kind (“✅ Safe to use by default”) is applicable when a reader can apply the tip without additional thinking. For example, [the `babel-plugin-lodash` tip](/README.md#enable-babel-plugin-lodash) is safe to use because a reader can just turn it on, and it won’t break anything.

The second kind (“⚠ Use with caution”) is applicable when using the tip requires additional thinking. For example, [the `lodash-webpack-plugin` tip](/README.md#enable-lodash-webpack-plugin) should be used with caution because blindly turning it on might break the app.

If you’re in doubt, ask us.

# 2. Write a tip using a template

Tips have a specific template. This helps readers extract the necessary information faster.

Generally, tips have the following template:

```markdown
### TIP NAME

> TIP KIND / HOW TO ENABLE / Added by [AUTHOR](AUTHOR LINK)

TIP TEXT
```

Here’s what each part means:

* `TIP NAME` is, well, the tip name. Fill it as you like.
* `TIP KIND` is either “✅ Safe to use by default” or “⚠ Use with caution”
* `HOW TO ENABLE` is either:
    * `[How to enable](LINK)` – a link to a page that describes how to enable the tip (e.g. a link to webpack plugin docs that describe enabling the plugin) or
    * `How to enable is ↓` if there’re no corresponding docs (in this case, describe the necessary steps in the text)
* `AUTHOR` and `AUTHOR LINK` is your name, twitter username, or whatever.
* `TIP TEXT` is the text of the tip.

# 3. Prepare for publishing

* Format the code to follow [Google JavaScript style guide](https://google.github.io/styleguide/jsguide.html) (specifically: use 2 spaces for indentation, add trailing commas, use camel space naming)
