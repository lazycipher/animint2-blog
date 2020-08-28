---
layout: post
title:  "Coding Period Report 2 (July 1, 2020 - July 30, 2020)"
author: lazycipher
categories: [ animint2, experience, gsoc ]
tags: [ featured ]
image: assets/images/post-heading/Coding-Period-Report-2.png
---

Hi, I'm Himanshu Singh(@lazycipher). In this post, I'll be sharing things that I've been doing from July 1 2020 to July 30 2020 i.e the second coding period.

#### What I did during the 2nd coding period?

I continued working on the testing infrastructure as there were few issues that were not being resolved earlier. I also removed examples which were meant to be for `ggplot2()` which were left while migrating the animint2 to use  ggplot2.
As one of the animint_kit_print was using knit print functions, I had to import knit_print as well and remove that from suggests. I tweaked the DESCRIPTION as per need. The `pt_to_lines function also needed tweaks to make it work.

- Animint2 now imports `knit_print` instead of suggests as it was needed for `animint_knit_print` to function.
- The new `grid::unitType()` function ([ref](https://developer.r-project.org/Blog/public/2020/04/13/changes-to-grid-units/index.html)).
    several packages were extracting attributes from “unit” objects, e.g., the "mm" from unit(1, "mm"); there is a new grid::unitType() function that may help packages to avoid accessing ‘grid’ unit internals in the future.
Currently we were extracting points unit type from unit objects to convert into lines inside the `pt_to_lines()` function but as it changed in grid package, we now use a new method provided by grid to check the type of unit.

```R
pt.to.lines <- function(pt_value){
  if(grid::unitType(pt_value) %in% c("pt", "points")){
    pt_value <- round(as.numeric(pt_value) * (0.25/5.5), digits = 2)
  }
  as.numeric(pt_value)
}
```

- Failed test because of not passing `stringsAsFactors = TRUE` in dataframes because of some recent changes: ([ref](https://developer.r-project.org/Blog/public/2020/02/16/stringsasfactors/index.html))
By detault, now, it's used `stringsAsFactors = FALSE` so we had to tweak some tests.

- Build failed everytime with the error shown below on the console of `Travis-ci` build.([ref](https://travis-ci.org/github/tdhock/animint2/builds/708614712))
After looking a lot and experinemting a lot, I found out that the secure github PAT was expired/invoked somehow and was causing this issue. I had to generate a new one and put the same in the `.travis.yml` file. So, the new secret makes the build run like a charm([ref](https://travis-ci.org/github/tdhock/animint2/builds/708651308)).
**Now, The test suite is working properly and all tests are passing.**
*Travis-ci error:*
```
Error in utils::download.file(url, path, method = method, quiet = quiet,  : 
  cannot open URL 'https://api.github.com/repos/ropensci/RSelenium/contents/DESCRIPTION?ref=v1.7.4'
Calls: <Anonymous> ... github_DESCRIPTION -> download -> base_download -> base_download_headers
Execution halted
The command "Rscript -e 'deps <- remotes::dev_package_deps(dependencies = NA);remotes::install_deps(dependencies = TRUE);if (!all(deps$package %in% installed.packages())) { message("missing: ", paste(setdiff(deps$package, installed.packages()), collapse=", ")); q(status = 1, save = "no")}'" failed and exited with 1 during .
```

- Updated the testing wiki documentation. Finally after everything was up and running, I updated the testing documentation. I also made two videos to show how it's done. It'll be helpful for new contributors. There's a dedicated [Videos Wiki page on Animint2](https://github.com/tdhock/animint2/wiki/Testing) repo which showcases all the videos.

- Moving `geom-*` specific code to `geom-*.R` class definitions for a better code readability and maintainability, I moved geom specific computation of data to geom specific class definition.
I'm working around this issue and I'll complete this very soon. As per a detailed discussion with Mr. Khan, everything look good till now and within a day or two, I'll start working on the next issue i.e moving the calculation of subset from compiler to renderer.

#### Any unsolved issue?

Yes, there are few things which are not completed as expected.

- The geom oop implementation was supposed to complete earlier but due to some issues it got delayed.
I'm working around this and will be completed in a day or two. So far it's going as expected as per Mr. Khan. ([ref](https://github.com/tdhock/animint2/pull/44))

- I'm working on few videos which shows how the animint2 works which will be good for new users to understand and use animint2.

#### Anything I messed up?

- I won't say that I messed up things but as I'm new to R programming ecosystem it took me some time in understnding the way animint2 works and that's why certain things are delayed as of now. I'll try to make things up and will cover up.

- Mr. Hock recommended me to make videos of how testing works and how animint2 works if possible. It was quite a hard time for me learning video editing I also had this fear of speaking but it ended well. Thanks a lot Mr. Hock for suggesting me to do this.
