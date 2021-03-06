name: webdrivercss
category: plugins
tags: guide
---

WebdriverCSS [![Build Status](https://travis-ci.org/webdriverio/webdrivercss.png?branch=master)](https://travis-ci.org/webdriverio/webdrivercss) [![Coverage Status](https://coveralls.io/repos/webdriverio/webdrivercss/badge.png?branch=master)](https://coveralls.io/r/webdriverio/webdrivercss?branch=master)
============

__CSS regression testing in WebdriverIO__. This plugin is an automatic regression-testing
tool for [WebdriverIO](http://webdriver.io). It was inspired by [James Cryers](https://github.com/jamescryer)
awesome project called [PhantomCSS](https://github.com/Huddle/PhantomCSS). After
initialization it enhances a WebdriverIO instance with an additional command called
`webdrivercss` and enables the possibility to save screenshots of specific parts of
your application.

### Never loose track of unwanted CSS changes:

![alt text](http://webdriver.io/images/webdrivercss/hero.png "Logo Title Text 1")


## How does it work?

1. Define areas within your application that should always look the same
2. Use WebdriverIO and WebdriverCSS to write some E2E tests and take screenshots of these areas
3. Continue working on your application or website
4. After a while rerun the tests
5. If desired areas differ from previous taken screenshots an image diff gets generated and you get notified in your tests


## Example

```js
// init WebdriverIO
var client = require('webdriverio').remote({desiredCapabilities:{browserName: 'chrome'}})
// init WebdriverCSS
require('webdrivercss').init(client);

client
    .init()
    .url('http://example.com')
    .webdrivercss('headerArea',{
        elem: '#header',
        screenWidth: [320,480,640]
    })
    .end();
```

## Install

WebdriverCSS uses GraphicsMagick/ImageMagick for image processing as well as [node-canvas](https://github.com/learnboost/node-canvas)
for comparing and analyzing screenshots with [node-resemble](https://github.com/kpdecker/node-resemble).
To install this package you'll need to have [GraphicsMagick](http://www.graphicsmagick.org/)
or [ImageMagick](http://www.imagemagick.org/) (one of them should be sufficient) and [Cairo](https://github.com/LearnBoost/node-canvas/wiki/_pages)
preinstalled.

### Mac OS X using [Homebrew](http://mxcl.github.io/homebrew/)
```sh
$ brew install imagemagick graphicsmagick cairo
```

### Ubuntu using apt-get (not tested)
```sh
$ sudo apt-get install imagemagick libmagickcore-dev
$ sudo apt-get install graphicsmagick
$ sudo apt-get install libcairo2-dev
```

### Windows

Download and install executables for [ImageMagick](http://www.imagemagick.org/script/binary-releases.php)/[GraphicsMagick](http://www.graphicsmagick.org/download.html)
and [Cairo](http://cairographics.org/download/).

After these dependencies are installed you can install WebdriverCSS via NPM as usual:

```sh
$ npm install webdrivercss
$ npm install webdriverio # if not already installed
```

## Setup

To use this plugin just call the `init` function and pass the desired WebdriverIO instance
as parameter. Additionally you can define some options to configure the plugin. After that
the `webdrivercss` command will be available only for this instance.

* **screenshotRoot** `String` ( default: *./webdrivercss* )<br>
  path were all screenshots gets saved

* **failedComparisonsRoot** `String` ( default: *./webdrivercss/diff* )<br>
  path were all screenshot diffs gets saved

* **misMatchTolerance** `Number` ( default: *0.05* )<br>
  number between 0 and 100 that defines the degree of mismatch to consider two images as
  identical, increasing this value will decrease test coverage

* **screenWidth** `Numbers[]` ( default: *[]* )<br>
  if set all screenshots will be taken in different screen widths (e.g. for responsive design tests)


### Example

```js
// create a WebdriverIO instance
var client = require('webdriverio').remote({
    desiredCapabilities: {
        browserName: 'phantomjs'
    }
});

// initialise WebdriverCSS for `client` instance
require('webdrivercss').init(client, {
    // example options
    screenshotRoot: 'my-shots';
    failedComparisonsRoot: 'diffs';
    misMatchTolerance: 0.05;
    screenWidth: [320,480,640,1024];
});
```

## Usage

WebdriverCSS enhances an WebdriverIO instance with an command called `webdrivercss`

`client.webdrivercss('some_id', {options}, callback);`

It provides options that will help you to define your areas exactly and exclude parts
that are unrelevant for design (e.g. content). Additionally it allows you to include
the responsive design in your regression tests easily. The following options are
available:

* **elem** `String`<br>
  only capture a specific DOM element

* **width** `Number`<br>
  define a fixed width for your screenshot

* **height** `Number`<br>
  define a fixed height for your screenshot

* **x** `Number`<br>
  take screenshot at an exact xy postion (requires width/height option)

* **y** `Number`<br>
  take screenshot at an exact xy postion (requires width/height option)

* **exclude** `String|Object`<br>
  exclude frequently changing parts of your screenshot, you can either pass a CSS3 selector
  that queries one or multiple elements or you can define x and y values which stretch
  a rectangle or polygon

* **excludeAfter** `Object`<br>
  exclude parts after taken screenshot

* **timeout** `Numbers`<br>
  wait a specific amount of time (in `ms`) to take the screenshot (useful if you have to wait
  on loading content)

The following paragraphs will give you a more detailed insight how to use these options properly.

### Let your test fail when screenshots differ

When using this plugin you can decide how to handle design breaks. You can either just work
with the captured screenshots or you could even break your E2E test at this position. The
following example shows how to handle design breaks within E2E tests:

```js
describe('my website should always look the same',function() {

    it('header should look the same',function(done) {
        client
            .url('http://www.example.org')
            .webdrivercss('header', {
                elem: '#header'
            }, function(err,res) {
                assert.equal(err, null);

                // this will break the test if screenshot differ more then 5% from
                // the previous taken image
                assert.equal(res.misMatchPercentage < 5, true);
            })
            .call(done);
    });

    // ...
```



### Define specific areas

The most powerful feature of WebdriverCSS is the possibility to define specific areas
for your regression tests. It is highly recommended to not just make screenshots of the
whole website. This can lead to many failing tests if someone breaks a tiny part of the
design. Additionally your tests will run slower because the CPU has to deal with bigger
images during the image comparison. It's better to have an own screenshot for each
UI component.

You can either use a CSS3 selector to define a DOM element you want to capture or you can
specify x/y coordinates to cover a more exact area.

```js
client
    .url('http://github.com')
    .webdrivercss('githubform', {
        elem: '#site-container > div.marketing-section.marketing-section-signup > div.container > form'
    });
```

Will capture the following:

![alt text](http://webdriver.io/images/webdrivercss/githubform.png "Logo Title Text 1")

**Tip:** do right click on the desired element, then click on `Inspect Element`, then hover
over the desired element in DevTools, open the context menu and click on `Copy CSS Path` to
get the exact CSS selector

The following example uses xy coordinates to capture a more exact area. You should also
pass a screenWidth option to make sure that your xy parameters map perfect on the desired area.

```js
client
    .url('http://github.com')
    .webdrivercss('headerbar', {
        x: 110,
        y: 15,
        width: 980,
        height: 34,
        screenWidth: [1200]
    });
```
![alt text](http://webdriver.io/images/webdrivercss/headerbar.png "Logo Title Text 1")




<h3>Exclude specific areas</h3>

Sometimes it is unavoidable that content gets captured and from time to time this content
will change of course. This would break all tests. To prevent this you can
determine areas, which will get covered in black and will not be considered anymore. Here is
an example:

```js
client
    .url('http://tumblr.com/themes')
    .webdrivercss('irgendwas', {
        exclude: '#theme_garden > div > section.carousel > div.carousel_slides,' +
                 '#theme_garden > div > section:nth-child(3) > div.theme_scroll_wrap,' +
                 '#theme_garden > div > section:nth-child(4) > div.theme_scroll_wrap',
        screenWidth: [1200]
    });
```
![alt text](http://webdriver.io/images/webdrivercss/exclude.png "Logo Title Text 1")

Instead of using a CSS3 selector you can also exclude areas by specifying xy values
which form a rectangle.

```js
client
    .url('http://tumblr.com/themes')
    .webdrivercss('irgendwas', {
        exclude: [{
            x0: 100, y0: 100,
            x1: 300, y1: 200
        }],
        screenWidth: [1200]
    });
```

If your exclude object has more then two xy variables, it will try to form a polygon. This may be
helpful if you like to exclude complex figures like:

```js
client
    .url('http://tumblr.com/themes')
    .webdrivercss('polygon', {
        exclude: [{
            x0: 120, y0: 725,
            x1: 120, y1: 600,
            x2: 290, y2: 490,
            x3: 290, y3: 255,
            x4: 925, y4: 255,
            x5: 925, y5: 490,
            x6: 1080,y6: 600,
            x7: 1080,y7: 725
        }],
        screenWidth: [1200]
    });
```
![alt text](http://webdriver.io/images/webdrivercss/exclude2.png "Logo Title Text 1")

<h3>Keep an eye on mobile screen resolution</h3>

It is of course also important to check your design in multiple screen resolutions. By
using the `screenWidth` option WebdriverCSS automatically resizes the browser for you.
By adding the screen width to the file name WebdriverCSS makes sure that only shots
with same width will be compared.

```js
client
    .url('http://stephencaver.com/')
    .webdrivercss('header', {
        elem: '#masthead',
        screenWidth: [320,640,960]
    });
```

This will capture the following image at once:

![alt text](http://webdriver.io/images/webdrivercss/header.new.960px.png "Logo Title Text 1")<br>
**file name:** header.960px.png

![alt text](http://webdriver.io/images/webdrivercss/header.new.640px.png "Logo Title Text 1")<br>
**file name:** header.640px.png

![alt text](http://webdriver.io/images/webdrivercss/header.new.320px.png "Logo Title Text 1")<br>
**file name:** header.320px.png
