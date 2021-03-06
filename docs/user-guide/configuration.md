# Configuration

The linter *expects a configuration object*.

Finding and loading of your configuration object is done with [cosmiconfig](https://github.com/davidtheclark/cosmiconfig). Starting from the current working directory, it will look for the following possible sources, in this order:

-   a `stylelint` property in `package.json`
-   a `.stylelintrc` file
-   a `stylelint.config.js` file exporting a JS object

The `.stylelintrc` file (without extension) can be in JSON or YAML format. Alternately, you can add a filename extension to designate JSON, YAML, or JS format:

-   `.stylelintrc.json`
-   `.stylelintrc.yaml` / `.stylelintrc.yml`
-   `.stylelintrc.js`.

You may want to use an extension so that your text editor can better interpret the file, and help with syntax checking and highlighting.

Once one of these is found and parsed, the search will stop and that object will be used. The configuration search can be short-circuited by using either the `config` or `configFile` options.

The configuration object can have the following properties.

## `rules`

Rules determine what the linter looks for and complains about. There are [over 170](rules/list.md) built into stylelint. *No rules are turned on by default*, so this is where you turn on everything you want to check. All the rules must be explicitly configured as *there are no default values*.

The `rules` property is *an object whose keys are rule names and values are rule configurations*. Each rule configuration fits one of the following formats:

-   a single value (the primary option)
-   an array with two values (`[primary option, secondary options]`)
-   `null` (to turn the rule off)

```json
{
  "rules": {
    "block-no-empty": null,
    "color-no-invalid-hex": true,
    "comment-empty-line-before": [ "always", {
      "ignore": ["stylelint-commands", "after-comment"]
    } ],
    "declaration-colon-space-after": "always",
    "indentation": ["tab", {
      "except": ["value"]
    }],
    "max-empty-lines": 2,
    "rule-empty-line-before": [ "always", {
      "except": ["first-nested"],
      "ignore": ["after-comment"]
    } ],
    "unit-whitelist": ["em", "rem", "%", "s"]
  }
}
```

Specifying a primary option will turn a rule on.

Many rules have secondary options which provide further customization. To set secondary options, a two-member array is used:

```js
"selector-pseudo-class-no-unknown": [true, {
  "ignorePseudoClasses": ["global"]
}]
```

### Turning rules off

To turn a rule off (when extending a configuration) you can set the value of the rule to `null`:

```json
{
  "extends": "stylelint-config-standard",
  "rules": {
    "at-rule-empty-line-before": null
  }
}
```

### Severities: error & warning

By default, all rules have an `"error"`-level severity. You can change this default by adding a `defaultSeverity` property to your configuration (see below).

To adjust any specific rule's severity, use the secondary option `severity`. The available values for `severity` are:

-   `"warning"`
-   `"error"`

```js
// error-level severity examples
{ "indentation": 2 }
{ "indentation": [2] }

// warning-level severity examples
{ "indentation": [2, { "severity": "warning" } ] }
{ "indentation": [2, {
    "except": ["value"],
    "severity": "warning"
  }]
}
```

Different reporters may use these severity levels in different way, e.g. display them differently, or exit the process differently.

### Custom Messages

If you want to deliver a custom message when a rule is violated, you can do so in two ways: provide a `message` option for the rule, or write a custom formatter.

All rules accept a `message` secondary option that, if provided, will be substituted for whatever standard message would be provided. For example, the following rule configuration would substitute in a couple of custom message:

```json
{
  "color-hex-case": [ "lower", {
    "message": "Lowercase letters are easier to distinguish from numbers"
  } ],
  "indentation": [ 2, {
    "except": ["block"],
    "message": "Please use 2 spaces for indentation. Tabs make The Architect grumpy.",
    "severity": "warning"
  } ]
}
```

Writing a [custom formatter](../developer-guide/formatters.md) gives you maximum control if you need serious customization.

## `extends`

Your configuration can *extend* an existing configuration (whether your own or a third-party config). When one configuration extends another, it starts with the other's properties then adds to and overrides what's there.

You can extend an array of existing configurations, with each item in the array taking precedence over the previous item (so the second item overrides rules in the first, the third item overrides rules in the first and the second, and so on, the last item overrides everything else).

For example, extending the [`stylelint-config-standard`](https://github.com/stylelint/stylelint-config-standard) and then changing indentation to tabs and turning off the `number-leading-zero` rule:

```json
{
  "extends": "stylelint-config-standard",
  "rules": {
    "indentation": "tab",
    "number-leading-zero": null
  }
}
```

Or starting with `stylelint-config-standard`, then layering `myExtendableConfig` on top of that, and then overriding the indentation rule:

```json
{
  "extends": [
    "stylelint-config-standard",
    "./myExtendableConfig"
  ],
  "rules": {
    "indentation": "tab"
  }
}
```

The value of `"extends"` is a "locater" (or an array of "locaters") that is ultimately `require()`d, so can fit whatever format works with Node's `require.resolve()` algorithm. That means a "locater" can be:

-   The name of a module in `node_modules` (e.g. `stylelint-config-standard`; that module's `main` file must be a valid JSON configuration)
-   An absolute path to a file (which makes sense if you're creating a JS object in a Node.js context and passing it in) with a `.js` or `.json` extension.
-   A relative path to a file with a `.js` or `.json` extension, relative to the referencing configuration (e.g. if configA has `extends: "../configB"`, we'll look for `configB` relative to configA).

*Because of `extends`, you can create and use shareable stylelint configurations.* Use the `stylelint-config` keyword within your `package.json` if publishing your config to npm.

## `plugins`

Plugins are rules or sets of rules built by the community that support methodologies, toolsets, *non-standard* CSS features, or very specific use cases.

Their *package* names are prefixed with "stylelint". Their *rule* names are namespaced so that they do not clash with stylelint's built-in rules.

Popular plugin packs include:

-   [`stylelint-order`](https://github.com/hudochenkov/stylelint-order): Specify the ordering of things e.g. properties within declaration blocks.
-   [`stylelint-scss`](https://github.com/kristerkari/stylelint-scss): Enforce a wide variety of SCSS-syntax specific linting rules.

You'll find more in [awesome stylelint](https://github.com/ntwb/awesome-stylelint).

To use one, add a `"plugins"` array to your config, containing "locaters" identifying the plugins you want to use. As with `extends`, above, a "locater" can be either an npm module name, an absolute path, or a path relative to the invoking configuration file.

Once the plugin is declared, within your `"rules"` object *you'll need to add options* for the plugin's rule(s), just like any standard rule. You will have to look at the plugin's documentation to know what the rule name should be.

```json
{
  "plugins": [
    "../special-rule.js"
  ],
  "rules": {
    "plugin/special-rule": "everything"
  }
}
```

A "plugin" can provide a single rule or a set of rules. If the plugin you use provides a set, invoke the module in your `"plugins"` configuration value, and use the rules it provides in `"rules"`. For example:

```json
{
  "plugins": [
    "../some-rule-set.js"
  ],
  "rules": {
    "some-rule-set/first-rule": "everything",
    "some-rule-set/second-rule": "nothing",
    "some-rule-set/third-rule": "everything"
  }
}
```

## `processors`

Processors are functions built by the community that hook into stylelint's pipeline, modifying code on its way into stylelint and modifying results on their way out. They enable stylelint to extract styles from within non-stylesheet files.

**Their use is discouraged as stylelint now has built-in support for many common non-stylesheet files.**

*Processors can only be used with the CLI and the Node.js API, not with the PostCSS plugin.* (The PostCSS plugin will ignore them.)

Processors can enable stylelint to lint, but not autofix, the CSS within non-stylesheet files that aren't supported out-of-the-box by stylelint.

To use one, add a `"processors"` array to your config, containing "locaters" identifying the processors you want to use. As with `extends`, above, a "locater" can be either an npm module name, an absolute path, or a path relative to the invoking configuration file.

```json
{
  "processors": ["stylelint-my-processor"],
  "rules": {..}
}
```

If your processor has options, make that item an array whose first item is the "locator" and second item is the options object.

```json
{
  "processors": [
    "stylelint-my-processor",
    [ "some-other-processor", { "optionOne": true, "optionTwo": false } ]
  ],
  "rules": {..}
}
```

## `defaultSeverity`

The default severity level for all rules that do not have a severity specified in their secondary options. The available values for `severity` are:

-   `"warning"`
-   `"error"`

## `ignoreFiles`

Provide a glob or array of globs to ignore specific files.

*Note that this is not an efficient method for ignoring lots of files.* If you want to ignore a lot of files efficiently, use `.stylelintignore` or adjust your files globs.

If the globs are absolute paths, they are used as is. If they are relative, they are analyzed relative to

-   `configBasedir`, if it's provided;
-   the config's filepath, if the config is a file that stylelint found a loaded;
-   or `process.cwd()`.

By default, all `node_modules` are ignored. Default values will be overridden if `ignoreFiles` is set.

The `ignoreFiles` property is stripped from extended configs: only the root-level config can ignore files.
