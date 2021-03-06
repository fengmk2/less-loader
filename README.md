# less loader for webpack

## Usage

[Documentation: Using loaders](http://webpack.github.io/docs/using-loaders.html)

``` javascript
var css = require("!raw!less!./file.less");
// => returns compiled css code from file.less, resolves imports
var css = require("!css!less!./file.less");
// => returns compiled css code from file.less, resolves imports and url(...)s
```

Use in tandem with the [`style-loader`](https://github.com/webpack/style-loader) to add the css rules to your document:

``` javascript
require("!style!css!less!./file.less");
```

### webpack config

``` javascript
module.exports = {
  module: {
    loaders: [
      {
        test: /\.less$/,
        loader: "style!css!less"
      }
    ]
  }
};
```

Then you only need to write: `require("./file.less")`

### LESS options

You can pass any LESS specific configuration options through to the render function via [query parameters](http://webpack.github.io/docs/using-loaders.html#query-parameters).

``` javascript
module.exports = {
  module: {
    loaders: [
      {
        test: /\.less$/,
        loader: "style!css!less?strictMath&noIeCompat"
      }
    ]
  }
};
```

See the [LESS documentation](http://lesscss.org/usage/#command-line-usage-options) for all available options. LESS translates dash-case to camelCase. Certain options which take values (e.g. `lessc --modify-var="a=b"`) are better handled with the [JSON loader syntax](http://webpack.github.io/docs/using-loaders.html#query-parameters) (`style!css!less?{"modifyVars":{"a":"b"}}`).  

### LESS plugins

In order to use [plugins](http://lesscss.org/usage/#plugins), simply define
the `lessLoader.lessPlugins` option. You can also change the options key
with a query parameter: `"less?config=lessLoaderCustom"`.

``` javascript
var LessPluginCleanCSS = require('less-plugin-clean-css');

module.exports = {
  ...
  lessLoader: {
    lessPlugins: [
      new LessPluginCleanCSS({advanced: true})
    ]
  }
};
```

## Imports

webpack provides an [advanced mechanism to resolve files](http://webpack.github.io/docs/resolving.html). The less-loader stubs less' `fileLoader` and passes all queries to the webpack resolving engine. Thus you can import your less-modules from `node_modules`. Just prepend them with a `~` which tells webpack to look-up the [`modulesDirectories`](http://webpack.github.io/docs/configuration.html#resolve-modulesdirectories)

```css
@import "~bootstrap/less/bootstrap";
```

It's important to only prepend it with `~`, because `~/` resolves to the home-directory. webpack needs to distinguish between `bootstrap` and `~bootstrap` because css- and less-files have no special syntax for importing relative files. Writing `@import "file"` is the same as `@import "./file";`

## Source maps

Because of browser limitations, source maps are only available in conjunction with the [extract-text-webpack-plugin](https://github.com/webpack/extract-text-webpack-plugin). Use that plugin to extract the CSS code from the generated JS bundle into a separate file (which even improves the perceived performance because JS and CSS are loaded in parallel).

Then your `webpack.config.js` should look like this:

```javascript
var ExtractTextPlugin = require('extract-text-webpack-plugin');

module.exports = {
    ...
    // must be 'source-map' or 'inline-source-map'
    devtool: 'source-map',
    module: {
        loaders: [
            {
                test: /\.less$/,
                loader: ExtractTextPlugin.extract(
                    // activate source maps via loader query
                    'css?sourceMap!' +
                    'less?sourceMap'
                )
            }
        ]
    },
    plugins: [
        // extract inline css into separate 'styles.css'
        new ExtractTextPlugin('styles.css')
    ]
};
```

If you want to view the original LESS files inside Chrome and even edit it,  [there's a good blog post](https://medium.com/@toolmantim/getting-started-with-css-sourcemaps-and-in-browser-sass-editing-b4daab987fb0). Checkout [test/sourceMap](https://github.com/webpack/less-loader/tree/master/test) for a running example. Make sure to serve the content with an HTTP server.

## Contribution

Don't hesitate to create a pull request. Every contribution is appreciated. In development you can start the tests by calling `npm test`.

The tests are basically just comparing the generated css with a reference css-file located under `test/css`. You can easily generate a reference css-file by calling `node test/helpers/generateCss.js <less-file-without-less-extension>`. It passes the less-file to less and writes the output to the `test/css`-folder.

[![build status](https://travis-ci.org/webpack/less-loader.svg)](https://travis-ci.org/webpack/less-loader)

## License

MIT (http://www.opensource.org/licenses/mit-license.php)
