---
layout: post
title:  "Test with Karma"
date:   2017-04-05 23:47:44
categories: Angular2-webpack-starter
---
# Test with Karma
- The entrance is `karmr.conf.js` in the root of the project
```js
// Look in ./config for karma.conf.js
module.exports = require('./config/karma.conf.js');
```
- Requires the configuration file in `config/karma.config.js`
# The config file
```js

module.exports = function (config) {
  var testWebpackConfig = require('./webpack.test.js')({ env: 'test' }); // run test after webpack packaging

  var configuration = {

    // base path that will be used to resolve all patterns (e.g. files, exclude)
    basePath: '', // refolve all files in the project

    /*
     * Frameworks to use
     *
     * available frameworks: https://npmjs.org/browse/keyword/karma-adapter
     */
    frameworks: ['jasmine'], // use jasmine which is suggested by Angular Team

    // list of files to exclude
    exclude: [],

    client: {
      captureConsole: false // disable all console output and pipe it to the terminal
    },

    /*
     * list of files / patterns to load in the browser
     *
     * we are building the test environment in ./spec-bundle.js
     */
    files: [
      { pattern: './config/spec-bundle.js', watched: false }, // prepare for 3rd packages and angular test env
      { pattern: './src/assets/**/*', watched: false, included: false, served: true, nocache: false } // do not test scan files in assets
    ],

    /*
     * By default all assets are served at http://localhost:[PORT]/base/
     */
    proxies: {
      "/assets/": "/base/src/assets/" // redirect static files access to assets folder
    },

    /*
     * preprocess matching files before serving them to the browser
     * available preprocessors: https://npmjs.org/browse/keyword/karma-preprocessor
     */
    preprocessors: { './config/spec-bundle.js': ['coverage', 'webpack', 'sourcemap'] }, // coverage for report, webpack for packaging, surcemap to link ts file and js file

    // Webpack Config at ./webpack.test.js
    webpack: testWebpackConfig,

    coverageReporter: {
      type: 'in-memory' // do not generate coverage report for compiled js files
    },

    remapCoverageReporter: { // generate coverage report with remap-istanbul
      'text-summary': null,
      json: './coverage/coverage.json',
      html: './coverage/html'
    },

    // Webpack please don't spam the console when running in karma!
    webpackMiddleware: {
      // webpack-dev-middleware configuration
      // i.e.
      noInfo: true,
      // and use stats to turn off verbose output
      stats: {
        // options i.e. 
        chunks: false
      }
    },

    /*
     * test results reporter to use
     *
     * possible values: 'dots', 'progress'
     * available reporters: https://npmjs.org/browse/keyword/karma-reporter
     */
    reporters: ['mocha', 'coverage', 'remap-coverage'],

    // web server port
    port: 9876,

    // enable / disable colors in the output (reporters and logs)
    colors: true,

    /*
     * level of logging
     * possible values: config.LOG_DISABLE || config.LOG_ERROR || config.LOG_WARN || config.LOG_INFO || config.LOG_DEBUG
     */
    logLevel: config.LOG_WARN,

    // enable / disable watching file and executing tests whenever any file changes
    autoWatch: false,

    /*
     * start these browsers
     * available browser launchers: https://npmjs.org/browse/keyword/karma-launcher
     */
    browsers: [
      'Chrome'
    ],

    customLaunchers: {
      ChromeTravisCi: {
        base: 'Chrome',
        flags: ['--no-sandbox']
      }
    },

    /*
     * Continuous Integration mode
     * if true, Karma captures browsers, runs the tests and exits
     */
    singleRun: true
  };

  if (process.env.TRAVIS) {
    configuration.browsers = [
      'ChromeTravisCi'
    ];
  }

  config.set(configuration);
};

```

# Npm scripts used for test
- test - npm run lint && karma start
    Run tslint and karma test with config file
    - singleRun
    - no auto-watch

![](/images/2017-04-06-00-16-37.jpg)

- watch:test - npm run test -- --auto-watch --no-single-run
    - auto-watch - override `singleRun: true` in config file
    - no-single-run - override `autoWatch: false` in config file

![](/images/2017-04-06-00-17-30.jpg)

# Webpack process
- webpack.test.js

# spec-bundle.js

# webpackMiddleware