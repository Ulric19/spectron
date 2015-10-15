# spectron

[![js-standard-style](https://img.shields.io/badge/code%20style-standard-brightgreen.svg?style=flat)](http://standardjs.com/)
[![Build Status](https://travis-ci.org/kevinsawicki/spectron.svg?branch=master)](https://travis-ci.org/kevinsawicki/spectron)
[![devDependencies:?](https://img.shields.io/david/kevinsawicki/spectron.svg)](https://david-dm.org/kevinsawicki/spectron)
<br>
[![license:mit](https://img.shields.io/badge/license-mit-blue.svg)](https://opensource.org/licenses/MIT)
[![npm:](https://img.shields.io/npm/v/spectron.svg)](https://www.npmjs.com/packages/spectron)
[![dependencies:?](https://img.shields.io/npm/dm/spectron.svg)](https://www.npmjs.com/packages/spectron)

Easily test your [Electron](http://electron.atom.io) apps using
[ChromeDriver](https://code.google.com/p/selenium/wiki/ChromeDriver) and
[WebdriverIO](http://webdriver.io).

This minor version of this library tracks the minor version of the Electron
versions released. So if you are using Electron `0.33.x` you would want to use
a `spectron` dependency of `^0.33` in your `package.json` file.

## Using

```sh
npm --save-dev spectron
```

Spectron works with any testing framework but the following example uses
[mocha](https://mochajs.org):

```js
var Application = require('spectron').Application
var assert = require('assert')

describe('application launch', function () {
  this.timeout(10000)

  beforeEach(function () {
    this.app = new Application({
      path: '/Applications/MyApp.app/Contents/MacOS/MyApp'
    })
    return this.app.start()
  })

  afterEach(function () {
    return this.app.stop()
  })

  it('shows an initial window', function () {
    return this.app.client.getWindowCount().then(function (count) {
      assert.equal(count, 1)
    })
  })
})
```

### With Chai As Promised

WebdriverIO is promise-based and so it pairs really well with the
[Chai as Promised](https://github.com/domenic/chai-as-promised) library that
builds on top of [Chai](http://chaijs.com).

Using these together allows you to chain assertions together and have fewer
callback blocks. See below for a simple example:

```sh
npm install --save-dev chai
npm install --save-dev chai-as-promised
```

```js
var Application = require('spectron').Application
var chai = require('chai')
var chaiAsPromised = require('chai-as-promised')
var path = require('path')

chai.should()
chai.use(chaiAsPromised)

describe('application launch', function () {
  beforeEach(function () {
    this.app = new Application({
      path: path: '/Applications/MyApp.app/Contents/MacOS/MyApp'
    })
    return this.app.start()
  })

  beforeEach(function () {
    chaiAsPromised.transferPromiseness = this.app.client.transferPromiseness
  })

  afterEach(function () {
    return this.app.stop()
  })

  it('opens a window', function () {
    return this.app.client.waitUntilWindowLoaded()
      .getWindowCount().should.eventually.equal(1)
      .isWindowMinimized().should.eventually.be.false
      .isWindowVisible().should.eventually.be.true
      .isWindowFocused().should.eventually.be.true
      .getWindowWidth().should.eventually.be.above(0)
      .getWindowHeight().should.eventually.be.above(0)
  })
})
```

### Application

#### new Application(options)

Create a new application with the following options:

* `path` - String path to the application executable to launch. **Required**
* `args` - Array of Chrome arguments to pass to the executable.
  See [here](https://sites.google.com/a/chromium.org/chromedriver/capabilities) for more details.
* `env` - Object of additional environment variables to set in the launched
  application.
* `host` - String host name of the launched `chromedriver` process.
  Defaults to `'localhost'`.
* `port` - Number port of the launched `chromedriver` process.
  Defaults to `9515`.
* `quitTimeout` - Number in milliseconds to wait for application quitting.
  Defaults to `1000` milliseconds.

#### start()

Starts the application. Returns a `Promise` that will be resolved when the
application is ready to use. You should always wait for start to complete
before running any commands.

#### stop()

Stops the application. Returns a `Promise` that will be resolved once the
application has stopped.

### Client Commands

Spectron uses [WebdriverIO](http://webdriver.io) and exposes the managed
`client` property on the created `Application` instances.

The full `client` API provided by WebdriverIO can be found
[here](http://webdriver.io/api.html).

Several additional commands are provided specific to Electron.

#### getWindowCount()

Gets the number of open windows.

```js
app.client.getWindowCount().then(function (count) {
  console.log(count)
})
```

#### getWindowDimensions()

Gets the dimensions of the current window. Object returned has
`x`, `y`, `width`, and `height` properties.

```js
app.client.getWindowDimensions().then(function (dimensions) {
  console.log(dimensions.x, dimensions.y, dimensions.width, dimensions.height)
})
```

#### getWindowHeight()

Get the height of the current window.

```js
app.client.getWindowHeight().then(function (height) {
  console.log(height)
})
```

#### getWindowWidth()

Get the width of the current window.

```js
app.client.getWindowWidth().then(function (width) {
  console.log(width)
})
```

#### isWindowDevToolsOpened()

Returns whether the current window's dev tools are opened.

```js
app.client.isWindowDevToolsOpened().then(function (devToolsOpened) {
  console.log(devToolsOpened)
})
```

#### isWindowFocused()

Returns whether the current window has focus.

```js
app.client.isWindowFocused().then(function (focused) {
  console.log(focused)
})
```

#### isWindowFullScreen()

Returns whether the current window is in full screen mode.

```js
app.client.isWindowFullScreen().then(function (fullScreen) {
  console.log(fullScreen)
})
```

#### isWindowLoading()

Returns whether the current window is loading.

```js
app.client.isWindowLoading().then(function (loading) {
  console.log(loading)
})
```

#### isWindowMaximized()

Returns whether the current window is maximized.

```js
app.client.isWindowMaximized().then(function (maximized) {
  console.log(maximized)
})
```

#### isWindowMinimized()

Returns whether the current window is minimized.

```js
app.client.isWindowMinimized().then(function (minimized) {
  console.log(minimized)
})
```

#### isWindowVisible()

Returns whether the current window is visible.

```js
app.client.isWindowVisible().then(function (visible) {
  console.log(visible)
})
```

#### setWindowDimensions(x, y, width, height)

Sets the window position and size.

```js
app.client.setWindowDimensions(100, 200, 50, 75)
```

#### waitUntilTextExists(selector, text, [timeout])

Waits until the element matching the given selector contains the given
text. Takes an optional timeout in milliseconds that defaults to `5000`.

```js
app.client.waitUntilTextExists('#message', 'Success', 10000)
```

#### waitUntilWindowLoaded([timeout])

Wait until the window is no longer loading. Takes an optional timeout
in milliseconds that defaults to `5000`.

```js
app.client.waitUntilWindowLoaded(10000)
```
