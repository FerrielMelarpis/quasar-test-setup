# Unit Test Setup

## Tech Stack
- karma
- mocha
- chai
- sinon
- vue-test-utils

## Unit Test Directory
- For more info about karma configuration, see http://karma-runner.github.io/1.0/config/configuration-file.html
```bash
mkdir -p test/unit/specs # this is where you put your test files <component>.spec.js
touch test/unit/{index.js,karma.conf.js}
```
- Structure
```
test/unit
├── coverage - don't need to create, it will be generated after running unit tests
├── index.js - initial file to run before running unit tests
├── karma.conf.js - karma config file
└── specs - contains unit test files <component>.spec.js
```

## [test/unit/index.js](../test/unit/index.js)
```js
require('jsdom-global')()

import Vue from 'vue'
import Quasar from 'quasar'


Vue.config.productionTip = false

Vue.use(Quasar);

// require all test files (files that ends with .spec.js)
const testsContext = require.context('./specs', true, /\.spec$/)
testsContext.keys().forEach(testsContext)

// require all files to be subjected under code coverage.
const srcContext = require.context(
  'src',
  true,
  /^\.\/((?!(main|router)(\.js)?$|data\/|config\/|statics\/|assets\/))/
)
srcContext.keys().forEach(srcContext)

```

## [test/unit/karma.conf.js](../test/unit/karma.conf.js)
```js
const webpackConfig = require('../../build/webpack.test.conf')

module.exports = function (config) {
	config.set({
		// to run in additional browsers:
		// 1. install corresponding karma launcher
		//    http://karma-runner.github.io/0.13/config/browsers.html
		// 2. add it to the `browsers` array below.
		browsers: ['PhantomJS'],
		frameworks: ['mocha', 'sinon-chai', 'phantomjs-shim'],
		reporters: ['spec', 'coverage'],
		files: ['./index.js'],
		preprocessors: {
		'./index.js': ['webpack', 'sourcemap'],
		},
		webpack: webpackConfig,
		webpackMiddleware: {
		noInfo: true
		},
		coverageReporter: {
		dir: './coverage',
		reporters: [
		{ type: 'lcov', subdir: '.' },
		{ type: 'text-summary' }
		],
			instrumenterOptions: {
		istanbul: { noCompact: true }
			}
		},
	})
}
```

## E2E Test Directory
- For more info on nightwatch configuration, see http://nightwatchjs.org
```bash
mkdir -p test/e2e/{custom-assertions,page-objects,reports,specs}
touch test/e2e/{nightwatch.conf.js,runner.js}
```
- Structure
```
t/e2e
├── custom-assertions - custom assertion methods
├── nightwatch.conf.js - nightwatch config file
├── page-objects - page objects, refer to nightwatch documentation
├── reports - don't actually need to create, it is created when reports are generated after running tests.
├── runner.js - test runner script
└── specs - contains test files <feature>.spec.js
```

## [test/e2e/nightwatch.conf.js](../test/e2e/nightwatch.conf.js)
```js
const config = require('../../config')

module.exports = {
    src_folders: ['test/e2e/specs'],
    output_folder: 'test/e2e/reports',
    custom_assertions_path: ['test/e2e/custom-assertions'],
    page_objects_path: ['test/e2e/page-objects'],

    selenium: {
        start_process: true,
        server_path: require('selenium-server').path,
        host: '127.0.0.1',
        port: 4444,
        cli_args: {
            'webdriver.chrome.driver': require('chromedriver').path
        }
    },

    test_runner: {
        type: 'mocha',
        options: {
            ui: 'bdd',
            reporter: 'spec'
        }
    },

    test_settings: {
        default: {
            selenium_port: 4444,
            selenium_host: 'localhost',
            silent: true,
            globals: {
                devServerURL: `http://localhost: ${process.env.PORT || config.dev.port}`
            }
        },

        chrome: {
            desiredCapabilities: {
                browserName: 'chrome',
                javascriptEnabled: true,
                acceptSslCerts: true
            }
        },

        firefox: {
            desiredCapabilities: {
                browserName: 'firefox',
                javascriptEnabled: true,
                acceptSslCerts: true
            }
        }
    }
}

```

## [test/e2e/runner.js](../test/e2e/runner.js)
```js

const server = require('../../build/script.dev.js')

server.ready.then(() => {

    let opts = process.argv.slice(2);
    if (opts.indexOf('--config') === -1) {
        opts = opts.concat(['--config', 'test/e2e/nightwatch.conf.js'])
    }
    if (opts.indexOf('--env') === -1) {
        opts = opts.concat(['--env', 'chrome'])
    }

    const spawn = require('cross-spawn');
    const runner = spawn('./node_modules/.bin/nightwatch', opts, { stdio: 'inherit' })

    runner.on('exit', function (code) {
        server.close()
        process.exit(code)
    })

    runner.on('error', function (err) {
        server.close()
        throw err
    })
})

```
