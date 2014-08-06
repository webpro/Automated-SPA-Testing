# Automated SPA Testing

## Introduction

This document represents some research done in order to find the best solution in client-side application testing, while meeting a set of requirements.

Many of the things described are implemented in this [baseplate](https://github.com/webpro/baseplate) project.

Please feel free to comment e.g. by opening tickets, or comment on this README.

## Contents

* [Types of Testing](#types-of-testing)
* [Requirements](#requirements)
* [Test Frameworks](#test-frameworks)
* [Assertion Libraries](#assertion-libraries)
* [Test Doubles](#test-doubles)
* [Test Runners](#test-runners)
    * [BusterJS](#busterjs)
    * [Intern](#intern)
    * [Mocha](#mocha)
    * [Karma](#karma)
    * [Testem](#testem)
    * [Functional Testing](#functional-testing)
    * [Comparison Table](#comparison-table)
    * [Performance](#performance)
* [Functional Testing - Alternative Solutions](#functional-testing---alternative-solutions)
* [CommonJS](#commonjs)

## Types of Testing

Some definitions of test type used here:

* **Unit Testing**: validate correctness of isolated units of the application.
* **Behavior Testing**: verify correct behavior of isolated functions based on user interaction, while faking events etc. in JavaScript.
* **Functional Testing**: verify correct behavior of functions, while the browser itself is driven by the tests (think Selenium, WebDriver API).
* **Integration Testing**: combine functions and involve real dependencies (CMS, database, etc.).
* **Acceptance Testing**: validate correct behavior of the product by the customer.

This document focuses on unit and behavior testing for client-side single-page applications (SPA), leaving functional, integration and acceptance testing out of scope.

## Requirements

* Specs can be written in JavaScript
* Support AMD setup (with RequireJS, maybe curl.js)
* BDD style is preferred over TDD
* Reuse tests across environments (e.g. browser, Node, PhantomJS)
* Quickly run tests from CLI without a real browser (e.g. Node, PhantomJS)
* Run the tests from CLI using real browsers installed on the system
    * Run tests in multiple browsers in a single run
    * Report back to CLI
    * Bonus points: Automatically open & close browser
* Include some and support extensible reporting formats (e.g. console, TAP)
* Bonus points: Output code coverage information

## Test Frameworks

Test frameworks provide the interface to write tests, run the tests, and report results.

* [Jasmine](http://pivotal.github.io/jasmine)
* [Mocha](http://visionmedia.github.io/mocha)

These frameworks support BDD style tests.

## Assertion Libraries

Assertion libraries provide the interface to make "assertions" (in TDD). They're called "expectations" in BDD.

* [chai](http://chaijs.com/)
* [expect.js](https://github.com/LearnBoost/expect.js)

Mocha leaves this open, and Jasmine ships with Matchers (`expect()` style methods). All libraries can be extended with custom assertions/matchers.

Note that Chai is a more popular choice and has more features, but does not support oldIE (while expect.js supports IE6+).

## Test Doubles

Since unit testing means testing isolated and dependency-free units, dependencies need to be mocked or stubbed out (i.e. "test doubles"). For instance, method calls to modules the unit depends on, and XHR calls to services.

* [Sinon.JS](http://sinonjs.org) is pretty much the de-facto standard here.
* [Jasmine has Spies](http://pivotal.github.io/jasmine/#section-Spies) built-in, but can also be [paired](https://github.com/froots/jasmine-sinon) with Sinon.JS.

## Test Runners

Test runners drive the test frameworks. Currently, popular solutions include:

* [BusterJS](http://busterjs.org)
* [Intern](http://theintern.io)
* [Karma](http://karma-runner.github.io)
* [Mocha](http://visionmedia.github.io/mocha)
* [Testem](https://github.com/airportyh/testem)

### BusterJS

Buster is a great effort and looks very interesting.

#### Issues

Configuration requires to load all files that _might_ be needed. E.g. `sources: ["src/**/*.js", "lib/**/*.js"]`. This might include a lot of unnecessary files (libraries may come with a large number of files that are not needed to use it), so it's not great performance wise. But the actual issue is that some matching `.js` files are Node executables, and thus start with a hash-bang (i.e. `#!/usr/bin/env node`), causing a fatal error for Buster (`[Fatal] Syntax error`).

Filed [issue #362](https://github.com/busterjs/buster/issues/362).

#### Project Activity

The project has an explicit "beta" stamp on it. Project activity seems pretty low ([Community](http://docs.busterjs.org/en/latest/community/), [Twitter](https://twitter.com/buster_js), [Issue Tracker](https://github.com/busterjs/buster/issues)).

Overall, this doesn't seem the right time to jump on this train (especially if things don't work straight away).

Of course I would love to see [things](https://github.com/webpro/baseplate) working!

### Intern

Intern is the new kid on the block, and it's quite impressive and complete. Many features come out of the box or are in the works, including Sauce Labs and Travis CI integration, functional testing (WebDriver API), and various code coverage reporters (istanbul).

### Karma

Karma is a _testacular_ effort from the AngularJS team, powered by Node and Socket.IO. It is fairly easy to set up. Configuration may result in some headaches for AMD setups (you want to reuse existing application and test configuration files). However, once it's done it doesn't need to be looked after anymore.

Karma comes with a decent set of reporters, and custom reporters can be plugged in.

For code coverage, istanbul is included (they pair up great). This is [easy to configure](http://karma-runner.github.io/0.8/config/coverage.html) with various output formats (including `html`, `lcov`, and `cobertura`).

### Mocha

Mocha is a test runner built with Node. It can run tests from CLI, but does not steer browsers.

However, it could still complement a setup where Mocha is the client test framework, and the test runner doesn't support running tests in Node. It _could_ be complementary, _if_ the Mocha test runner would support AMD. And this _could_ be solved by using PhantomJS, if Mocha didn't have [this issue](https://github.com/visionmedia/mocha/issues/770) since v1.10. However, things work using [mocha-phantomjs](http://metaskills.net/mocha-phantomjs/) and Mocha v1.9.

Mocha supports code coverage output out of the box. It can also be configured to work with:

* [istanbul](https://github.com/yahoo/istanbul)
* [Blanket.js](http://blanketjs.org)
* [JSCoverage](http://siliconforks.com/jscoverage)

### Testem

Testem is built with Node (read more about it in [Testem: Interactive JS Test Runner](http://tobyho.com/2012/06/24/testem-interactive-js-test-runner/)).

While configuration is not without hurdles in the other test runners, Testem has an advantage by the ability to hook into a [custom test page](https://github.com/airportyh/testem#custom-test-pages). When a page is configured for testing (i.e. with Jasmine or Mocha), that page needs two extra lines of code (to inject `testem.js`). Then, if Testem is driving the show, it automatically detects the framework in the page, and hooks into it (i.e. with a custom reporter for Jasmine, or monkey-patching Mocha's `Runner.prototype.emit`).

In short, if Jasmine or Mocha runs, Testem runs.

Additionally, a big plus of this strategy is that it's easy to configure functional testing (see below).

A downside of Testem is that it doesn't have support for code coverage (at least not built-in). Since Testem hooks into the client side frameworks, those can be configured to use coverage reporters (but after generating coverage files from sources). One working effort includes [Testem Coverage Sandbox](https://github.com/richardbutler/testem-coverage-sandbox).

### Behavior Testing

Based on the given definition ("verify correct behavior of isolated functions based on user interaction, while faking events etc. in JavaScript"), the challenge is to initialize a function of the application, and then simulate user interaction with it. Examples include:

* Navigate the UI by clicking links or buttons, and display associated content.
* Fill out a form and submit it by using the keyboard.
* Simulate different ways  of interaction (i.e. mouse, keyboard, touch)

Things are actually quite feasible, even simple, when extending the workflow and tools for unit testing to functional tests. What is needed on top of this:

* Fixtures for initial DOM structures (to simulate e.g. pre-rendered HTML from CMS)
* Ability to simulate user interaction such as clicks, key presses, etc.

There are a couple of caveats when programmatically simulating user interactions:

* Filling out an input field (e.g. `$('input[name=firstName]').val('John')`) won't trigger a "change" event. This can be simulated by appending `.trigger('change')`, or use something like [jQuery.autotype](https://github.com/mmonteleone/jquery.autotype) (untested).
* Simulating touch events is possible by dispatching [Touch Events](http://www.w3.org/TR/touch-events/), or use something like [fake-touches](https://github.com/jtangelder/faketouches.js) (untested).

It's actually trivial to set this up using Testem, Jasmine (or Mocha) and jQuery. Driven by Testem, the functional tests can easily be ran cross-browser and/or quickly using only PhantomJS. Here is [how it might look like](https://github.com/webpro/baseplate/blob/master/test/mocha-behavior/specs/moduleB.behavior.js) in a working example.

It should be feasible to do this with Karma as well (untested).

### Comparison Table

Features | BusterJS | Intern | Mocha (1) | Karma | Testem |
:--|:-:|:-:|:-:|:-:|:-:
AMD | Y ([2](http://docs.busterjs.org/en/latest/extensions/buster-amd/)) | Y | N ([3](http://metaskills.net/mocha-phantomjs)) | Y (2) | Y
BDD | Y | Y | Y | Y | Y
Jasmine | N ([3](https://github.com/mattfysh/jasmine-buster)) | N | N (2) | Y (2) | Y
Mocha | N | N | Y | Y (2) | Y
CLI: Node | Y | Y | N (4)
CLI: PhantomJS | Y | Y ([2](https://github.com/theintern/intern/wiki/Using-Intern-with-PhantomJS)) | N ([3](http://metaskills.net/mocha-phantomjs)) | Y | Y
CLI: Run tests in real browser | Y | Y | N | Y | Y
CLI: Open/close browser | N | Y (5) | N | Y | Y
Code coverage | | Y | | Y | N
Behavior Tests | Y | N | | Y | Y
Functional Tests | N | Y | | N | N

1. This is the Mocha test runner (Mocha client test framework is a feature)
1. Feature included (e.g. as extension/adapter), but may require extra configuration.
1. Feature available through external project.
1. Mocha _uses_ Node, but not suited for AMD setup.
1. Through Sauce Labs integration (not local).

### Performance

Running a single test, including opening and closing 4 different browsers is fast in Karma and Testem. Testem has the option to run browsers in parallel, while Karma seems to do this automatically. The (same) test is run using minimal, default settings; Jasmine; and in Chrome, Chrome Canary, Firefox and PhantomJS:

Command | Time (real)
:-- | ---
`time testem ci -P 4` | 2.8s
`time karma start --single-run` | 3.8s

Not sure how things stack up on other systems, in other browsers and with more tests.

## Functional Testing

For functional testing, when the browser itself needs to be driven by the tests (e.g. for page navigation, file uploads), solutions like the following might be interesting to check out:

* [CasperJS](http://casperjs.org/) - Functional testing solution on top of CasperJS (i.e. not in real browsers)
* [Selenium WebDriver](http://docs.seleniumhq.org/projects/webdriver/) - API to automate testing web applications (available in [many languages](http://docs.seleniumhq.org/docs/03_webdriver.jsp), using [various browser drivers](https://code.google.com/p/selenium/wiki/FrequentlyAskedQuestions#Q:_Which_browsers_does_WebDriver_support?))
* [WD.js](https://github.com/admc/wd) - WebDriver/Selenium 2 for Node ([used by Intern](https://github.com/theintern/intern#features))

Note: The [WebDriver API](http://www.w3.org/TR/webdriver/) is a W3C standard (which in turn is based on Selenium WebDriver).

## CommonJS

Source code can be written in CommonJS, and be wrapped in the AMD transporter format for the browser. In doing so, some things around testing would probably get easier. Especially configuration and running stuff in Node (think Mocha) come to mind. The Mocha test runner drives CommonJS-style modules/specs. Interesting if real browser testing is out of scope, and e.g. speed and code coverage reporting options are more important.

Switching to CommonJS source and spec files doesn't seem of much benefit to Karma and Testem (apart from ease of configuration). Both Karma and Testem can pre-process source files if needed (i.e. convert to AMD).

## Conclusion

Please, draw your own! YMMV.
