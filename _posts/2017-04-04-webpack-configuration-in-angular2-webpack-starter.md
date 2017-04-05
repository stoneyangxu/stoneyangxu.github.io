---
layout: post
title:  "Webpack configuration in Angular2-webpack-starter"
date:   2017-04-04 19:15:49
categories: Angular2-webpack-starter
---
# The configuration in root directory `webpack.config.js`
```js
switch (process.env.process.env.NODE_ENV) {
  case 'prod':
  case 'production':
    module.exports = require('./config/webpack.prod')({env: 'production'});
    break;
  case 'test':
  case 'testing':
    module.exports = require('./config/webpack.test')({env: 'test'});
    break;
  case 'dev':
  case 'development':
  default:
    module.exports = require('./config/webpack.dev')({env: 'development'});
}
```

- depends on the `NODE_ENV` value, webpack decides which config file to work with.
- `webpack.prod.js` - is used for real production 
- `webpack.test.js`- is used for test
- `webpack.dev.js` - is used for develop

## `require('./config/webpack.prod')` returns a function that returns webpack config and use `{env: 'production'}` as parameter

![](/images/2017-04-04-20-35-01.jpg)

- `commonConfig` returns common config object shared by prod, dev and test
- [`webpackMerge`](https://www.npmjs.com/package/webpack-merge) is an 3rd lib to merge webpack configs
> webpack-merge provides a merge function that concatenates arrays and merges objects creating a new object. If functions are encountered, it will execute them, run the results through the algorithm, and then wrap the returned values within a function again
- the second parameter for webpackMerge is the configs for `specified env`

# Webpack Common Config

## Ahead of all, check package mode by reading `node process env and argument`

```js
const HMR = helpers.hasProcessFlag('hot');  // for webpack hmr
const AOT = helpers.hasNpmFlag('aot'); // for Angular2 Aot
const METADATA = {
  title: 'Angular2 Webpack Starter by @gdi2290 from @AngularClass', 
  baseUrl: '/', 
  isDevServer: helpers.isWebpackDevServer() // run with webpack-dev-server
};
```
- `hasProcessFlag` - check whether `flag` exists in the npm command as `an argument`
```js
function hasProcessFlag(flag) {
  return process.argv.join('').indexOf(flag) > -1;
}
```
- `hasNpmFlag` - npm_lifecycle_event is the script name which is running and `hasNpmFlag` will check weather the script name contains `flag`
```js
const EVENT = process.env.npm_lifecycle_event || '';
function hasNpmFlag(flag) {
  return EVENT.includes(flag);
}
```
- `isWebpackDevServer` - checks whether to run with webpack-dev-derver

## cache - `true` as default

## entry - The entry point for the bundle
```js

    /*
     * The entry point for the bundle
     * Our Angular.js app
     *
     * See: http://webpack.github.io/docs/configuration.html#entry
     */
    entry: {

      'polyfills': './src/polyfills.browser.ts',
      'main':      AOT ? './src/main.browser.aot.ts' : './src/main.browser.ts'
    },
```

- polyfills - create bundle base on packages imported in `polyfills.browser.ts`
- main - create bundle for the main application, use different file depends on `AOT`
    - main.browser.ts use `bootstrapModule` to load `AppModule`
    - main.browser.aot.ts use `bootstrapModuleFactory` to load `AppModuleNgFactory` which is compiled

## resolve - Options affecting the resolving of modules

```js

    /*
     * Options affecting the resolving of modules.
     *
     * See: http://webpack.github.io/docs/configuration.html#resolve
     */
    resolve: {

      /*
       * An array of extensions that should be used to resolve modules.
       *
       * See: http://webpack.github.io/docs/configuration.html#resolve-extensions
       */
      extensions: ['.ts', '.js', '.json'],

      // An array of directory names to be resolved to the current directory
      modules: [helpers.root('src'), helpers.root('node_modules')],
    },
```

- resolve '.ts', '.js', '.json' files in src and node_modules
- `helpers.root` function is used to get real path 

```js

var path = require('path');
var ROOT = path.resolve(__dirname, '..');
var root = path.join.bind(path, ROOT);
exports.root = root;
```

## module - Options affecting the normal modules

### Rules for `ts` files

- [@angularclass/hmr-loader](https://github.com/AngularClass/angular2-hmr) - Hot Module Reloading for Webpack 2 and Angular 2

```js
              loader: '@angularclass/hmr-loader',
              options: {
                pretty: !isProd,
                prod: isProd
              }
            },
```

- [ng-router-loader](https://www.npmjs.com/package/ng-router-loader) - Webpack loader for NgModule lazy loading using the angular router. Supports AOT and hot module load.

```js
            { // MAKE SURE TO CHAIN VANILLA JS CODE, I.E. TS COMPILATION OUTPUT.
              loader: 'ng-router-loader',
              options: {
                loader: 'async-import',
                genDir: 'compiled',
                aot: AOT
              }
            },
```

- [awesome-typescript-loader](https://www.npmjs.com/package/awesome-typescript-loader) - compile ts files to js

```js
            {
              loader: 'awesome-typescript-loader',
              options: {
                configFileName: 'tsconfig.webpack.json'
              }
            }
```

- [angular2-template-loader](https://www.npmjs.com/package/angular2-template-loader) - inlines all html and style's in angular2 components 

```js
            {
              loader: 'angular2-template-loader'
            }
```

### Other loaders 

- json-loader
- css-loader
- sass-loader
- raw-loader for htmls - import files as a string
- file-loader for images in css 
> notice `exclude` in css, sass and raw-loader, they will be loaded inline with `angular2-template-loader`

## plugins - Add additional plugins to the compiler

### assets-webpack-plugin - Webpack plugin that emits a json file with assets paths

> This plug-in outputs a json file with the paths of the generated assets

### CheckerPlugin - `CheckerPlugin` is optional. Use it if you want async error reporting

### CommonsChunkPlugin - get common codes from multiple chunks

```js
      new CommonsChunkPlugin({
        name: 'polyfills',
        chunks: ['polyfills']
      }),
      // This enables tree shaking of the vendor modules
      new CommonsChunkPlugin({
        name: 'vendor',
        chunks: ['main'],
        minChunks: module => /node_modules/.test(module.resource)
      }),
      // Specify the correct order the scripts will be injected in
      new CommonsChunkPlugin({
        name: ['polyfills', 'vendor'].reverse()
      }),
```

- The chunk name of the commons chunk. An existing chunk can be selected by passing a name of an existing chunk
- `minChunks: module => /node_modules/.test(module.resource)` - move 3rd packages into vender

### ContextReplacementPlugin - Provides context to Angular's use of System.import
> https://github.com/angular/angular/issues/11580

- replace `/angular(\\|\/)core(\\|\/)src(\\|\/)linker/,` to `/angular(\|/)core(\|/)@angular/,` for angular 4

### CopyWebpackPlugin - copy files

```js
      new CopyWebpackPlugin([
        { from: 'src/assets', to: 'assets' },
        { from: 'src/meta'}
      ]),
```

### HtmlWebpackPlugin - inject metadata and bundles into html file

```js
      new HtmlWebpackPlugin({
        template: 'src/index.html',
        title: METADATA.title,
        chunksSortMode: 'dependency',
        metadata: METADATA,
        inject: 'head'
      }),
```

### ScriptExtHtmlWebpackPlugin - add defer to each script

```js
      new ScriptExtHtmlWebpackPlugin({
        defaultAttribute: 'defer'
      }),
```

### HtmlElementsPlugin - Generate html tags based on javascript maps

```js
      new HtmlElementsPlugin({
        headTags: require('./head-config.common')
      }),
```

- add tags in html head

```html
  <!-- Configured Head Tags  -->
  <link rel="apple-touch-icon" sizes="57x57" href="/assets/icon/apple-icon-57x57.png">
	<link rel="apple-touch-icon" sizes="60x60" href="/assets/icon/apple-icon-60x60.png">
	<link rel="apple-touch-icon" sizes="72x72" href="/assets/icon/apple-icon-72x72.png">
	<link rel="apple-touch-icon" sizes="76x76" href="/assets/icon/apple-icon-76x76.png">
	<link rel="apple-touch-icon" sizes="114x114" href="/assets/icon/apple-icon-114x114.png">
	<link rel="apple-touch-icon" sizes="120x120" href="/assets/icon/apple-icon-120x120.png">
	<link rel="apple-touch-icon" sizes="144x144" href="/assets/icon/apple-icon-144x144.png">
	<link rel="apple-touch-icon" sizes="152x152" href="/assets/icon/apple-icon-152x152.png">
	<link rel="apple-touch-icon" sizes="180x180" href="/assets/icon/apple-icon-180x180.png">
	<link rel="icon" type="image/png" sizes="192x192" href="/assets/icon/android-icon-192x192.png">
	<link rel="icon" type="image/png" sizes="32x32" href="/assets/icon/favicon-32x32.png">
	<link rel="icon" type="image/png" sizes="96x96" href="/assets/icon/favicon-96x96.png">
	<link rel="icon" type="image/png" sizes="16x16" href="/assets/icon/favicon-16x16.png">
	<link rel="manifest" href="/assets/manifest.json">
	<meta name="msapplication-TileColor" content="#00bcd4">
	<meta name="msapplication-TileImage" content="/assets/icon/ms-icon-144x144.png">
	<meta name="theme-color" content="#00bcd4">
```

### LoaderOptionsPlugin 
> The UglifyJsPlugin no longer puts loaders into minimize mode. The debug option has been removed. Loaders should no longer read their options from the webpack configuration. Instead you need to provide these options with the LoaderOptionsPlugin

### NormalModuleReplacementPlugin - replace resource

```js
      new NormalModuleReplacementPlugin(
        /facade(\\|\/)async/,
        helpers.root('node_modules/@angular/core/src/facade/async.js')
      ),
      new NormalModuleReplacementPlugin(
        /facade(\\|\/)collection/,
        helpers.root('node_modules/@angular/core/src/facade/collection.js')
      ),
      new NormalModuleReplacementPlugin(
        /facade(\\|\/)errors/,
        helpers.root('node_modules/@angular/core/src/facade/errors.js')
      ),
      new NormalModuleReplacementPlugin(
        /facade(\\|\/)lang/,
        helpers.root('node_modules/@angular/core/src/facade/lang.js')
      ),
      new NormalModuleReplacementPlugin(
        /facade(\\|\/)math/,
        helpers.root('node_modules/@angular/core/src/facade/math.js')
      ),
```

### [ngcWebpack.NgcWebpackPlugin](https://www.npmjs.com/package/ngc-webpack) - Angular Template Compiler Wrapper for Webpack

```js
      new ngcWebpack.NgcWebpackPlugin({
        disabled: !AOT,
        tsConfig: helpers.root('tsconfig.webpack.json'),
        resourceOverride: helpers.root('config/resource-override.js')
      })
```

# Prod Config VS Dev Config

## Get metadata from common config and replace variables with webpackMerge

- Appending `host`, `port` and `ENV`
- Only enable `hor module replacement` for develop and with `hot` flag

![](/images/2017-04-05-21-15-44.jpg)

## devtool - Choose a developer tool to enhance debugging

- Use different devtool for prod and dev:

![](/images/2017-04-05-21-17-06.jpg)

![](/images/2017-04-05-21-11-18.jpg)

## path - The output directory as absolute path (required).

- build result files to dist 
- they are the same in prod and dev
```js
      path: helpers.root('dist'),
```
## filenames
- filename - name of the bundle
- sourceMapFilename - name of the source map file
- chunkFilename - name of non-entry files
-  `chunkhash` id added in prod

![](/images/2017-04-05-21-18-34.jpg)

## .ts loader
- `tslint-loader` is added in dev

![](/images/2017-04-05-21-22-47.jpg)

- There is not any new rules for ts files in prod. Using the rules in common config, only load and compile.

## css and sass loader
- css and sass used by Angular Components (exclude: [helpers.root('src', 'styles')]) have already loaded in common config, 
- the other style files are `loaded`(include: [helpers.root('src', 'styles')])
- the difference between prod and dev is that styles are `compressed with ExtractTextPlugin` in prod

![](/images/2017-04-05-21-35-22.jpg)
## Optimize JS and Extract CSS files is additional in prod
```js
      /**
       * Webpack plugin to optimize a JavaScript file for faster initial load
       * by wrapping eagerly-invoked functions.
       *
       * See: https://github.com/vigneshshanmugam/optimize-js-plugin
       */

      new OptimizeJsPlugin({
        sourceMap: false
      }),

      /**
       * Plugin: ExtractTextPlugin
       * Description: Extracts imported CSS files into external stylesheet
       *
       * See: https://github.com/webpack/extract-text-webpack-plugin
       */
      new ExtractTextPlugin('[name].[contenthash].css'),

```

## Define variables with metadata

```js
      new DefinePlugin({
        'ENV': JSON.stringify(METADATA.ENV),
        'HMR': METADATA.HMR,
        'process.env': {
          'ENV': JSON.stringify(METADATA.ENV),
          'NODE_ENV': JSON.stringify(METADATA.ENV),
          'HMR': METADATA.HMR,
        }
      }),

```

## Use UglifyJsPlugin to uglify js files `in prod`
```js

      /**
       * Plugin: UglifyJsPlugin
       * Description: Minimize all JavaScript output of chunks.
       * Loaders are switched into minimizing mode.
       *
       * See: https://webpack.github.io/docs/list-of-plugins.html#uglifyjsplugin
       */
      // NOTE: To debug prod builds uncomment //debug lines and comment //prod lines
      new UglifyJsPlugin({
        beautify: false, //prod
        output: {
          comments: false
        }, //prod
        mangle: {
          screw_ie8: true
        }, //prod
        compress: {
          screw_ie8: true,
          warnings: false,
          conditionals: true,
          unused: true,
          comparisons: true,
          sequences: true,
          dead_code: true,
          evaluate: true,
          if_return: true,
          join_vars: true,
          negate_iife: false // we need this for lazy v8
        },
      }),
```

## Replace some module `in prod`
```js
      new NormalModuleReplacementPlugin(
        /angular2-hmr/,
        helpers.root('config/empty.js')
      ),

      new NormalModuleReplacementPlugin(
        /zone\.js(\\|\/)dist(\\|\/)long-stack-trace-zone/,
        helpers.root('config/empty.js')
      ),
```
## LoaderOptionsPlugin
- Use LoaderOptionsPlugin to define special options form `htmlLoader` in prod
- Only set debug to be true in dev
```js
          htmlLoader: {
            minimize: true,
            removeAttributeQuotes: false,
            caseSensitive: true,
            customAttrSurround: [
              [/#/, /(?:)/],
              [/\*/, /(?:)/],
              [/\[?\(?/, /(?:)/]
            ],
            customAttrAssign: [/\)?\]?=/]
          },

```
# Special configs in dev
## library and libraryTarget
```js
      library: 'ac_[name]',
      libraryTarget: 'var',
```
- output.library allows you to optionally specify the name of your library.

- output.libraryTarget allows you to specify the type of output. I.e. CommonJs, AMD, for usage in a script tag or as UMD module.

## DllBundlesPlugin
> A Plugin for Webpack that uses Webpack's DllPlugin & DllReferencePlugin to create bundle configurations as part of the build process. The plugin will monitor for changes in packages and rebuild the bundles accordingly

![](/images/2017-04-05-21-59-53.jpg)

## AddAssetHtmlPlugin
- add dll files into html file
```js
      /**
       * Plugin: AddAssetHtmlPlugin
       * Description: Adds the given JS or CSS file to the files
       * Webpack knows about, and put it into the list of assets
       * html-webpack-plugin injects into the generated html.
       *
       * See: https://github.com/SimenB/add-asset-html-webpack-plugin
       */
      new AddAssetHtmlPlugin([
        { filepath: helpers.root(`dll/${DllBundlesPlugin.resolveFile('polyfills')}`) },
        { filepath: helpers.root(`dll/${DllBundlesPlugin.resolveFile('vendor')}`) }
      ]),
```

## devServer - start a server for development
```js
    /**
     * Webpack Development Server configuration
     * Description: The webpack-dev-server is a little node.js Express server.
     * The server emits information about the compilation state to the client,
     * which reacts to those events.
     *
     * See: https://webpack.github.io/docs/webpack-dev-server.html
     */
    devServer: {
      port: METADATA.port,
      host: METADATA.host,
      historyApiFallback: true,
      watchOptions: {
        aggregateTimeout: 300,
        poll: 1000
      }
    },
```
