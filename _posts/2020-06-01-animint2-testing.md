---
layout: post
title:  "Animint2 Testing Documentation"
author: lazycipher
categories: [ animint2, testing, docs ]
image: assets/images/post-heading/animint2-testing-docs.png
---

Testing Animint2: If you're making changes in animint2 and want test if everything is working properly, you need to make sure all the tests written in `animint2/tests/testthat/*` passes.
Whenever you make a PR or push any commit, it gets tested using Travis-CI but it's not best way to test your code by pushing code everytime to github and check for errors on Travis-CI. Animint2 provides you a way to run the tests on your system and even see what's being rendered on browser.
In this article we'll see how we can get the testing suite up and running and get our tests done.

#### Requirements

Before starting, you need the following packages installed on your system.

- Animint2(using `R CMD INSTALL`)
- [RSelenium@1.7.4](https://github.com/ropensci/RSelenium/releases/tag/v1.7.4)
- Docker(If you want to test on browser)
- Selenium Docker Image([selenium/standalone-firefox-debug:2.53.0](https://hub.docker.com/layers/selenium/standalone-firefox-debug/2.53.0/images/sha256-37ab374c61ee2477d5d56510d6c845a1005401c9d89cd9434b6a2a908d4f6c2c?context=explore))
- [PhantomJS](https://github.com/ariya/phantomjs/releases/tag/2.1.1)(If testing in headless mode)

#### Installing Animint2

If you made any changes in R code, you might want to build the package from source to see your changes in package.
In order to do that, simply go to the directory where `animint2` source code is available and run the following command.

Go to the directory where animint2 is inatalled for ex:
if it's `/media/lazycipher/projects/animint2` then you've to goto `/media/lazycipher/projects/` and then run the following command.

```R
R CMD INSTALL animint2
```

#### Starting selenium docker image(for real browser testing)

<mark>Note: You don't need this for headless testing.</mark>

Pull the image if not already pulled:

```
docker pull selenium/standalone-firefox-debug:2.53.0
```

Now start the same:

**For Linux**:

```
docker run -d --network="host" selenium/standalone-firefox-debug:2.53.0
```

**For Windows and Mac**:

```
docker run -d -p 4444:4444 -p 5900:5900 selenium/standalone-firefox-debug:2.53.0
```

You can navigate to `http://localhost:4444/wd/hub/static/resource/hub.html` to see if selenium is running.


#### Start R in `animint2/tests` directory:

You need to start R session inside `animint2/tests` directory or just use `setwd()` to change working directory to `tests` directory.

Once you've started R session inside tests directory, copy paste the following in R console:

```R
library("testthat")
library("animint2")
library("RSelenium");library("XML")
setwd("testthat")
source("helper-functions.R")
source("helper-HTML.R")
source("helper-plot-data.r")
```

It'll load the needed libraries and helper functions.

#### Initialize remote web driver

<mark>For headless browser testing</mark> we use phantomjs. You need to run the following to start webdriver and start phantomjs binary.

```R
tests_init()
```

It'll start phantomjs binary and start remote driver for the same.

<mark>For real browser test(firefox in our case)</mark>, just pass `"firefox"` in the function. So, It'll be:

```R
tests_init("firefox")
```

It'll start a firefox browser instance on the selenium server standalone.

#### Running tests

We've the function `tests_run()` which will run the tests and we can pass `filter=""` in order to test only specific tests. Let's see how it works:

```R
# to run all tests
tests_run() 
# to run specific
tests_run(filter="axis-rotate")
# to run all renderer tests
tests_run(filter="renderer")
```

<mark>If using Firefox for testing</mark> you might want to see what's being rendered on browser so, open a **VNC** client and connect to the url `localhost:5900` or `127.0.0.1:5900`
If it asks for password, it's by default **`secret`**.

If you're feeling stuck or facing issues feel free to let us know by [creating issues](https://github.com/tdhock/animint2/issues).
