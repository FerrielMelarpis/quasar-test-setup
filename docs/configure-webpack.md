This is the most probable reason why you got stuck with setting up a test
in your quasar project. Quasar has a different format for configuring webpack.

## [script.dev.js](../build/script.dev.js)
- We're just reusing the dev server for running our e2e tests and to do that, we need to remove
the manual setting of `process.env.NODE_ENV`.
```
-process.env.NODE_ENV = 'development'
```

- We're going to change the export of this module.
```
-module.exports = app.listen(port, function (err) {
+console.log('> Starting server...')
+var ready = new Promise(function (resolve) {
+  devMiddleware.waitUntilValid(function () {
+    console.log('> Listening at ' + uri + '\n')
+    // open only on dev env
+    if (config.dev.openBrowser && process.env.NODE_ENV === 'development') {
+      opn(uri)
+    }
+
+    resolve()
+  })
+})
+
+var server = app.listen(port, function (err) {
   if (err) {
     console.log(err)
     process.exit(1)
   }
+})

-  // open browser if set so in /config/index.js
-  if (config.dev.openBrowser) {
-    devMiddleware.waitUntilValid(function () {
-      opn(uri)
-    })
+module.exports = {
+  server,
+  ready,
+  close: function () {
+    server.close()
   }
-})
+}
```

## [webpack.test.conf.js](../build/webpack.test.conf.js)
- Install `html-loader`.
- Now, we need to create the test configuration for webpack
```js
var webpack = require('webpack')
var merge = require('webpack-merge')
var HtmlWebpackPlugin = require('html-webpack-plugin')

var cssUtils = require('./css-utils')
var baseWebpackConfig = require('./webpack.base.conf')

var webpackConfig = merge(baseWebpackConfig, {
  module: {
    rules: cssUtils
      .styleRules({
        sourceMap: true,
        postcss: true
      })
      .concat([{
        test: /\.html$/,
        loader: 'html-loader'
      }])
  },
  // use inline sourcemap for karma-sourcemap-loader
  devtool: '#inline-source-map',
  resolveLoader: {
    alias: {
      /**
       * Necessary to to make lang="scss" work when using vue-loader's ?inject option
       * The issue with this is that lang="scss" actually needs to use sass-loader,
       * so when it goes to load it can't find scss-loader as that is not a loader available.
       * see discussion at https://github.com/vuejs/vue-loader/issues/724
       * for more info: https://webpack.github.io/docs/loaders.html
       */
      'scss-loader': 'sass-loader'
    }
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env': require('../config/test.env')
    }),
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'src/index.html',
      inject: true
    })
  ]
})

// no need for app entry during tests
delete webpackConfig.entry

module.exports = webpackConfig
```

## [config/test.env.js](../config/test.env.js)
- And we also need a test env config
```js
module.exports = {
  NODE_ENV: '"test"'
}
```
