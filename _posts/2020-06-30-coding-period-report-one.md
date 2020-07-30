---
layout: post
title:  "Coding Period Report 1 (June 1, 2020 - June 30, 2020)"
author: lazycipher
categories: [ animint2, experience, gsoc ]
tags: [ featured ]
image: assets/images/post-heading/Coding-Period-Report-1.png
---

Hi, I'm Himanshu Singh(@lazycipher). In this post, I'll be sharing things that I've been doing from the day when coding period of GSoC'20 started.

## What I did during the first month of `coding period`?

The very first issue was with the code testing. The versions of packages used were depricated so I had to make sure everything was updated and was up and running. So, during these two weeks I worked on making the test suite work with latest versions of needed testing frameworks and other dependencies. The tools which are used in the suite are testthat and RSelenium which further requires binaries of selenium and a browser instance of either firefox or phantomjs to render the generated webpages and run tests.

In order to make everything up and running, I decided to upgrade all dependencies and solve issues one by one. While doing this, I did a huge mistake which I'll mention later in this post. The very first thing was that previous used functions to start a selenium binary and start a web driver on the top of that were deprecated. So, I went through the documentation and tweaked as per need.

Below was the earlier written code to start binaries as per need:

```R
if (browserName == "phantomjs") {
    message("Starting phantomjs binary. To shut it down, run: \n pJS$stop()")
    pJS <<- RSelenium::phantom()
  } else {
    message("Starting selenium binary. To shut it down, run: \n",
            "remDr$closeWindow() \n",
            "remDr$closeServer()")
    RSelenium::checkForServer(dir = system.file("bin", package = "RSelenium"))
    selenium <- RSelenium::startServer()
  }
remDr <<- RSelenium::remoteDriver(browserName = browserName, ...)
```
In this, RSelenium's `phantom()`, `checkForServer()`, `startServer()` were deprecated. So I used 
the recommended functions for the replacement.

The recommended way was to use `rsDriver()` function which was supposed to run a selenium binary and then start a web driver on the top of that. <mark>It worked fine on windows but due to some issues It was making R crash in Linux based systems.</mark> Apart from that I kept getting <mark>selenium and firefox version inincompatibility.</mark> The version of selenium which R used was v2 and the firefox version compatible with that was deprecated one so I decided to solve this and make it more new contributor friendly. 

I decided to move the testing suite use docker so that the used browser and selenium instances couldn't affect user's system. I couldn't find any way so I decided to make it work other way round. I decided finally to run a docker image on user's computer and connect to that usung `remoteDriver()` function.

There were few issues that I faced while implementing this to work with windows, linux and mac systems.

The very first issue was that, we were using a server where we kept rendering and replacing different html files needed for the specific tests. It was the `animint-htmltest` directory inside the `./tests/testthat/` directory.
```R
run_servr(port = port, directory = testPath)
```
<mark>The server created was running on user's system which was supposed to be accessed from the browser instance created inside the docker.</mark> This was first time I was using docker in my life so it took time for me to figure out the way to do that. 

The recommended way was to use `docker.host.internal` in mac or windows systems but similar hack like this wasn't implemented yet for linux but there was another jugaad(hack) for that which was to pass `--network="host"` so we could access everything from docker to system or vice-versa.
So at the end if the user's system was linux, animint_server was supposed to be accessible on `localhost` while `docker.host.internal` for windows and mac systems.

At the end, the new code segment looked something like this-

```R
OS <- Sys.info()[['sysname']]
    if(OS == "Linux") {
      animint_server <- "localhost"
    }
    if(OS == "Windows" || OS == "Darwin") {
      animint_server <- "host.docker.internal"
    }

  if (browserName == "phantomjs") {
    message("Starting phantomjs binary. To shut it down, run: \n pJS$stop()")
    
    pJS <<- wdman::phantomjs(
                  port = 4444L,
                  phantomver = "latest"
                )
    # Give time for phantomjs binary to start
    animint_server <- "localhost"
    Sys.sleep(8)  
  } else {
    
    # If using firefox, you'll need to run selenium-firefox docker image in order to make it work correctly.
    # We're using docker to avoid version incompatibility issues.
    message("You need to run selenium docker image(selenium/standalone-firefox:2.53.0) as specified in docs(). \nNote: Ignore if already running.")
  }
  
  remDr <<- RSelenium::remoteDriver(
    port = 4444L,
    browser = browserName,
  )

```

And the testing suite worked fine!


<mark>I also fixed the issue with units used in `margin()` function as mentioned here:</mark> [/issues/31](https://github.com/tdhock/animint2/issues/31).

The issue was because recently, `grid` used a new unit implementation. The margin function created it's own unit objects which were incompatible with the new unit implementation.
Earlier it was:
```R
margin <- function(t = 0, r = 0, b = 0, l = 0, unit = "pt") {
  structure(unit(c(t, r, b, l), unit), class = c("margin", "unit"))
}
```
Which now is replaced with:
```R
margin <- function(t = 0, r = 0, b = 0, l = 0, unit = "pt") {
  u <- unit(c(t, r, b, l), unit)
  class(u) <- c("margin", class(u))
  u
}
```

#### Any unsolved issue?

I think the new testing suite with upgraded functions is already up and running. But there are certain issues-
- with Inf/-Inf rendering on graph. see: [issue comment](https://github.com/tdhock/animint2/issues/38#issuecomment-650785386)
- Unnecessary gaps between multiple facets and extra margin below graph title. see: [/issues/40](https://github.com/tdhock/animint2/issues/40)

I'm working on these and I believe it'll cause delay in the proposed timeline.

- Tests running on `travis` stuck on the main repository while passes on the forked one. 
See: [Main](https://travis-ci.org/github/tdhock/animint2) and [Forked](https://travis-ci.org/github/lazycipher/animint2/builds/702910153).
Error:
```
The downloaded source packages are in
	‘/tmp/Rtmp1Uc0xD/downloaded_packages’
2.32s$ Rscript -e 'deps <- remotes::dev_package_deps(dependencies = NA);remotes::install_deps(dependencies = TRUE);if (!all(deps$package %in% installed.packages())) { message("missing: ", paste(setdiff(deps$package, installed.packages()), collapse=", ")); q(status = 1, save = "no")}'
Error in utils::download.file(url, path, method = method, quiet = quiet,  : 
  cannot open URL 'https://api.github.com/repos/ropensci/RSelenium/contents/DESCRIPTION?ref=v1.7.4'
Calls: <Anonymous> ... github_DESCRIPTION -> download -> base_download -> base_download_headers
Execution halted
The command "Rscript -e 'deps <- remotes::dev_package_deps(dependencies = NA);remotes::install_deps(dependencies = TRUE);if (!all(deps$package %in% installed.packages())) { message("missing: ", paste(setdiff(deps$package, installed.packages()), collapse=", ")); q(status = 1, save = "no")}'" failed and exited with 1 during .
Your build has been stopped.
```
 Until we find anything helpful on this, I'm supposed to be working on the forked version of repository.
 
#### Anything I messed up?

Yes, Since it was my first time interacting with R ecosystem, I overlooked one thing. <mark>It happened before my step of moving to docker as a solution.</mark> I assumed that the latest version of `RSelenium` supported `Selenium v3` atleast as v4 is in alpha now.
Because of this, few functions were not working as expected. One was the `clickElement()` and other was `sendKeysToActiveElement()`. Talking about the first function, It never threw any error on console so I couldn't understand the reason behind this issue. But I found a jugaad that was `executeScript()` function of RSelenium which was supposed to execute JavaScript in browser. I decided to <mark>use event dispatcher</mark> and pass a click using this.

It Worked!

So basically, I tweaked previously defined `clickID()` function to use the following code:
```R
remDr$executeScript(sprintf("document.getElementById('%s').dispatchEvent(new CustomEvent('click'))", as.character(id)))
```

Everything was going fine until I moved to fix the `sendKeysToActiveElement()` function. Firstly I tried solving this using JS but later on looking on the error and inspecting bit more, <mark>when I executed this function manually, I got an error stating that this was an unsupported command or method.</mark>
I looked around and found that this was deprecated in Selenium v3. I was hit hard to see that as I realized I used an incompatible version of selenium. Later on, I found on the github repo of rselenium that they supported v2 only!
<mark>Then I decided to use docker to avoid any such kind of version issues in future in this project.</mark>

That's all happened during this first month, Apart from that, I learned many new things about testing, CI and team collaboration.
