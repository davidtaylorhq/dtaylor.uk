+++
title = "Creating a QUnit test harness for headless Chrome"
slug = "chrome-qunit"
date = 2017-08-20T12:04:19+01:00
thumbnail = ""
tags = ["discourse", "testing"]
description = "Making use of Puppeteer to create a CLI QUnit runner"
toc = true
+++

When running acceptence tests in a continuous integration system, we often need some way to simulate a browser. For some time, [PhantomJS](http://phantomjs.org/) has been the go-to option, thanks to its simple API and lightweight nature. 

Chrome 59 recently launched with support for ["headless mode"](https://developers.google.com/web/updates/2017/04/headless-chrome):

> It's a way to run the Chrome browser in a headless environment. Essentially, running Chrome without chrome! It brings all modern web platform features provided by Chromium and the Blink rendering engine to the command line.

Soon after this announcment, came the news that PhantomJS would [cease to be maintained](https://groups.google.com/d/msg/phantomjs/9aI5d-LDuNE/5Z3SMZrqAQAJ):

> Headless Chrome is coming

> I think people will switch to it, eventually. Chrome is faster and more stable than PhantomJS. And it doesn't eat memory like crazy.

The [Discourse](https://discourse.org) project currently uses PhantomJS to run its QUnit tests, so I decided to see what a switch to headless Chrome would look like. The resultant NPM module is not tied to Discourse, and should work for any project utilising QUnit.

## Taking control of Chrome
The first thing to realise is that headless Chrome does not offer any 'new' way of controlling the browser programmatically. Essentially all the `--headless` flag does is remove the need for a screen (or virtual screen) to display the user interface on.

Fortunately, Chrome already has a built-in solution for programmatic control in the form of its [Chrome Devtools Protocol](https://chromedevtools.github.io/devtools-protocol/). Starting Chrome with the `--remote-debugging-port` flag opens up a WebSocket based protocol for debugging. 

The development of client libraries has exploded since the launch of headless chrome, with a number of them 'trending' on GitHub at the time of writing. These seem to fall into two categories: those with low-level APIs mapping closely to the available features (e.g. [chrome-remote-interface](https://github.com/cyrus-and/chrome-remote-interface), and those with higher-level APIs to easier write functional tests (e.g. [Chromeless](https://github.com/graphcool/chromeless)). 

Discourse already utilises QUnit for writing the tests themselves, so a simpler, lower-level API should suffice. I chose the [puppeteer](https://github.com/GoogleChrome/puppeteer) library because it was created and is maintained by the same team that maintain the Chromium DevTools interface. It also includes a bundled version of Chromium, guarenteed to be compatible with the included API. This comes in at just under 100mb, which is significantly less than the ~450mb required to install the full version of Chrome on Ubuntu.

With Node.js 8 installed, Puppeteer can be installed simply by running
```shell
npm install puppeteer
```

To try it out, make a javascript file `test.js` containing 

```javascript
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('https://dtaylor.uk');
  await page.screenshot({path: 'example.png'});

  browser.close();
})();
```
and execute by running `node test.js`.


## Creating a harness for QUnit tests

Let's assume that there is already a server running which serves the QUnit tests at `localhost:3000/qunit`.

The basic logic of the harness is as follows:

1. Load the QUnit test page
2. Attach to QUnit's events to log results for each test
3. Once finished, print a summary
4. Exit with an appropriate exit code for success/failure

### Basic setup
Because our tests will be running inside Chrome, we can't directly see the error logs. To pass them through to the node logs, we attach to the 'console' event on the page object:
```javascript
await page.on('console', (...params) => {
  for (let i = 0; i < params.length; ++i)
    console.log(`${params[i]}`);
});
```
### Browser > harness communication
QUnit has four levels of event we want to capture: an individual assertion, a test, a module, and the whole test suite. We can get callbacks for these three events using `QUnit.log`, `QUnit.testDone`, `QUnit.moduleDone` and `QUnit.done`. 

Puppeteer allows us to register functions under the top level `window` object in the browser, to allow us to send information from the browser back to our harness.

```javascript
await page.exposeFunction('harness_log', context => {
  if (context.result) { return; } // If assertion successful don't log
  
  // Logic here to log result of an assertion
});
```

Similar functions are setup for `testDone` and `moduleDone`, check the full source code for the details.

Notably, these functions are attached to the `window` object as `async` functions. This means that when called they immediately return a Promise, and do not block the code in the browser. This is good for performance, but means we cannot use them directly as callback functions in QUnit - we need to wrap them inside another 'synchronous' function.

### Running the tests

Now we have communication from the browser to the harness, we're ready to load the page. Once it's loaded, some code is run to set any necessary QUnit variables and attach the QUnit event handlers.

```javascript
await page.evaluate(() => {
  QUnit.config.testTimeout = 10000;

  QUnit.moduleDone((context) => { window.harness_moduleDone(context); });
  QUnit.testDone((context) => { window.harness_testDone(context); });
  QUnit.log((context) => { window.harness_log(context); });
  QUnit.done((context) => { window.harness_done(context); });

  console.log("Running: " + JSON.stringify(QUnit.urlParams));
});
```

After this, we simply wait for the `done` callback. If the timeout is reached, the process is ended with a failure code:

```
function wait(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
await wait(timeout);

console.error("Tests timed out");
browser.close();
process.exit(124);
```

### Success?

When QUnit is done, the `harness_done` function is called. This prints out a summary, and finally exits with an appropriate status code:

```javascript
await page.exposeFunction('harness_done', context => {
  if (moduleErrors.length > 0) {
    for (var idx=0; idx<moduleErrors.length; idx++) {
      console.error(moduleErrors[idx]+"\n");
    }
  }

  var stats = [
    "Time: " + context.runtime + "ms",
    "Total: " + context.total,
    "Passed: " + context.passed,
    "Failed: " + context.failed
  ];
  console.log(stats.join(", "));
  
  browser.close();
  if (context.failed > 0){
    process.exit(1);
  }else{
    process.exit();
  }
});
```

## Great, how do I use it?
Simply run 
```shell
npm install -g qunit-puppeteer
puppeteer-qunit http://localhost:3000/qunit
```

The output will look something like this:
```plain
Running: {}

............................................................................................................................................................................

Time: 27157ms, Total: 173, Passed: 173, Failed: 0
```

Source code for the project can be found [on GitHub](https://github.com/davidtaylorhq/qunit-puppeteer)


