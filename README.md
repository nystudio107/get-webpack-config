# get-webpack-config

Utilities to help you modularize your webpack configs

## About get-webpack-config
`get-webpack-config` is a set of lightweight utilities to help you modularize your webpack configs.

You can break up your webpack configs into small modular configs that are easier to understand & maintain, and then merge them together for webpack.

It also allows you to have optional settings that can be passed in to each config, allowing you to update the configs independently of the settings.

## get-webpack-config example

Here's an example of how you might use `get-webpack-config`, this is a complete webpack config file for a development server:

```js
// webpack.dev.js
// developmental build config

// environment
process.env.NODE_ENV = process.env.NODE_ENV || 'development';

// webpack config file helpers
const { modernWebpackConfigs } = require('get-webpack-config');

// development module exports
module.exports = modernWebpackConfigs(
    'app',
    'dev-server',
    'manifest',
    'babel-loader',
    'image-loader',
    'font-loader',
    'postcss-loader',
    'typescript-loader',
    'vue-loader',
);
```

You'd run it just as you'd run any webpack build, e.g.:

```
webpack serve --config webpack.dev.js
```

This would look for webpack configs in the default `webpack-configs/` directory given names, with `.config.js` appended to them.

It will also look for corresponding settings in the default `webpack-settings/` directory, and pass them in to each config.

Then using webpack-merge, it merges the configs together, and passes them back to webpack.

Here's an example of what the directory structure would look like, with each `.config.js` and optional `.settings.js` file:

```
├── package.json
├── package-lock.json
├── webpack.dev.js
├── webpack-configs
│   ├── app.config.js
│   ├── babel-loader.config.js
│   ├── dev-server.config.js
│   ├── font-loader.config.js
│   ├── image-loader.config.js
│   ├── manifest.config.js
│   ├── postcss-loader.config.js
│   ├── typescript-loader.config.js
│   └── vue-loader.config.js
└── webpack-settings
    ├── app.settings.js
    ├── babel-loader.settings.js
    ├── dev-server.settings.js
    ├── manifest.settings.js
    └── typescript-loader.settings.js
```

## webpack-configs

A webpack config for `get-webpack-config` is exactly the same as any webpack config: it's a `.js` file that returns a [webpack configuration object](https://webpack.js.org/configuration/).

The webpack config is passed in the `type` of build (either `modern` or `legacy`), and a `settings` object. Modern builds are intended for modern browsers, and legacy builds are intended for legacy browsers, using the [module/nomodule pattern](https://philipwalton.com/articles/deploying-es2015-code-in-production-today/)

Here's an example of an empty example config:

```js
// example.config.js
// returns a webpack config object for the example config

// return a webpack config
module.exports = (type = 'modern', settings) => {
    // common config
    const common = () => ({
    });
    // configs
    const configs = {
        // development configs
        development: {
            // legacy development config
            legacy: {
            },
            // modern development config
            modern: {
                ...common(),
            },
        },
        // production configs
        production: {
            // legacy production config
            legacy: {
                ...common(),
            },
            // modern production config
            modern: {
                ...common(),
            },
        }
    };

    return configs[process.env.NODE_ENV][type];
}
```

Note that the above is just a _convention_, all the default export needs to do is return a webpack config object.

Following this convention, though, allows you to easily create webpack configs that are different depending on their `type` and whether it's a development or production build.

The `common()` function contains a common webpack config that's spread into the actual configs, then each specific config can add in whatever makes it unique.

**N.B.:** The `development`/`legacy` config combo is empty, because it's rare to want to use legacy builds in a HMR development environment.

## webpack-settings

The `.settings.js` files are just JavaScript modules that export an object of config settings.

Here's an example:

```js
// example.settings.js

// node modules
require('dotenv').config();

// settings
module.exports = {
    foo: 'bar',
};

```

We're `require()`'ing the optional [dotenv package](https://www.npmjs.com/package/dotenv) because it's common to want to override settings from `.env` files.

All you actually need to do is ensure that the default export returns an object of settings that your config can use.

The settings are kept separate from the config files so that webpack configs can be updated independent of the settings that affect them.

There is a special settings file `app.settings.js` that is merged with all other settings files, so you can put global settings in there.

## Exported functions

For all of the functions below, whether to return a production or development config is based on `process.env.NODE_ENV`.

### `modernWebpackConfigs(configName, ...)`

This function takes an arbitrary number of config names as arguments, and returns a merged webpack config from them all.

It ends up calling `getModernWebpackConfig()` on each argument, to return the modern version of the config.

### `legacyWebpackConfigs(configName, ...)`

This function takes an arbitrary number of config names as arguments, and returns a merged webpack config from them all.

It ends up calling `getLegacyWebpackConfig()` on each argument, to return the legacy version of the config.

### `buildWebpackConfigs(configName, ...)`

This function takes an arbitrary number of config names as arguments, and returns a merged webpack config from them all.

It ends up calling `getModernWebpackConfig()` on each argument, to return the modern version of the config.

This function exists just as a way to logically separate out any webpack tasks that are not related to either the modern or legacy builds, such as "clean" or "copy" tasks.

### `getModernWebpackConfig(configName)`

This function returns a single webpack config, by passing in the corresponding `.settings.js` to it (if any) and returning the results.

It appends `.config.js` to `configName` to determine the name of the config to load.

It returns the modern version of the config.

### `getLegacyWebpackConfig(configName)`

This function returns a single webpack config, by passing in the corresponding `.settings.js` to it (if any) and returning the results.

It appends `.config.js` to `configName` to determine the name of the config to load.

It returns the legacy version of the config.

### `getWebpackSettings(settingsName)`

This function returns a settings object for the passed in `settingsName`.

It appends `.settings.js` to `configName` to determine the name of the settings to load.

## Environment variables

The following environment variables allow you to override defaults in `get-webpack-config`:

- `GET_WEBPACK_CONFIG_BASE_CONFIG_NAME` - the name of the global base settings file that should be merged into any webpack config specific settings. Default: `app`

- `GET_WEBPACK_CONFIG_SETTINGS_PATH` - The fully qualified path to the directory where the webpack configs are. Default: `./webpack-configs`

- `GET_WEBPACK_CONFIG_CONFIGS_PATH` - The fully qualified path to the directory where the webpack settings are. Default: `./webpack-settings`
