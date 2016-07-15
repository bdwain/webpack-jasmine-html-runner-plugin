# webpack-jasmine-html-runner-plugin

> *interactive in-browser testing* with webpack.

* allows using [webpack](http://webpack.github.io/) & its dev-server for in-browser [Jasmine](http://jasmine.github.io/2.4/introduction.html) tests,

* generates a `spec-runner.html` webpack entry for each of your JavaScript spec-files, specified by a pattern,

* uses [sourcemapped-stacktrace](https://github.com/novocaine/sourcemapped-stacktrace) to fix stack traces in the Jasmine HTML reporter.


### Installation

First make sure that `webpack` and `webpack-dev-server` are installed and configured for your project.
Make sure that Jasmine 2 is installed in your project, and that your spec-tests use the Jasmine APIs (`describe`, `beforeEach`, etc.) as global functions.

```console
npm install --save-dev webpack-jasmine-html-runner-plugin
```


### Usage

Create a webpack configuration for your tests, e.g. `webpack.spec.config.js`.

In this configuration:

```js
const webpack = require( 'webpack' );
const WebpackJasmineHtmlRunnerPlugin = require( 'webpack-jasmine-html-runner-plugin' );

// load or otherwise create your base webpack configuration:
const config = require( './webpack.config' );

// in this example, your spec-runners are named e.g. 'lib/some-module/spec/spec-runner.js'
config.entry = WebpackJasmineHtmlRunnerPlugin.entry( './lib/[name]/spec/spec-runner.js' );
config.output = {
   path: path.resolve( path.join( process.cwd(), 'spec-output' ) ),
   publicPath: '/spec-output/',
   filename: '[name].bundle.js'
};

config.plugins = [
   new webpack.SourceMapDevToolPlugin( {
      filename: '[name].bundle.js.map'
   } ),
   new WebpackJasmineHtmlRunnerPlugin( {
      fixupScripts: []
   } )
];
```

Now, run webpack and the webpack-dev-server:

```console
webpack --config webpack.spec.config.js
webpack-dev-server --inline --config webpack.spec.config.js
```

You may skip the first step (running `webpack`), but doing so allows you to conveniently click through a file listing in your web browser, to navigate to you spec-test.

When the webpack dev server has finished setting up, you can visit your spec-tests under [localhost:8080/webpack-dev-server/spec-output/](http://localhost:8080/webpack-dev-server/spec-output/) (append `some-module/spec-runner.html` to view the test for `some-module`).


### Usage with fixup scripts

To improve debugging convenience, the plugin comes with several *fixup scripts* that augment the Jasmine 2 HTML reporter.
They can be controlled using the option `fixupScripts`, which takes an array of strings:

* `fixup-stacktraces`: uses [sourcemapped-stacktrace](https://github.com/novocaine/sourcemapped-stacktrace) to improve Jasmine stacktraces by looking them up against a source map (if available).

* `fixup-json-messages`: improves formatting of Jasmine error reports when using `expect( objectA ).toEqual( objectB )`.
  The builtin error messages are relatively hard to read. When using this script, the JSON objects are reformatted, and you can click on them to get a nice colored diff generated by [jsondiffpatch](https://github.com/benjamine/jsondiffpatch).

If you remove the configuration `fixupScripts: []` from the example above, you revert to the default of `[ 'fixup-stacktraces', 'fixup-json-messages' ]`.


### Usage with additional script

Let's assume that your project or its tests need additional polyfills or framework script to be loaded.
You could try to shoehorn it into the webpack entry points generated above, but the `WebpackJasmineHtmlRunnerPlugin` also has an option for this:

Create a file `polyfills.js` containing your additional dependencies and modify your configuration like this:

```js
const config = require( /* see above */ );
config.entry = WebpackJasmineHtmlRunnerPlugin.entry( /* see above */ );
config.output = { /* see above */ };

// add a webpack entry to also generate polyfills
config.entry.polyfills = './polyfills.js';

config.plugins = [
   new webpack.SourceMapDevToolPlugin( {
      filename: '[name].bundle.js.map'
   } ),
   new WebpackJasmineHtmlRunnerPlugin( {
      // add an include path to load the polyfills
      includePaths: [ path.resolve( process.cwd(), 'spec-output/polyfills.bundle.js' ) ]
   } )
];
```
