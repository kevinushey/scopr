
Later
=====

Scoped side effects and signals for R.

[![Travis-CI Build Status](https://travis-ci.org/kevinushey/later.svg?branch=master)](https://travis-ci.org/kevinushey/later)

This package:

-   generalizes `on.exit()` to a function `defer()`, which can attach exit handlers to any currently executing function, and uses that to scope side effects, and

-   provides a simple signal / event system, using event handlers registered through `on()` / `off()`, and events fired through `signal()`.

Scoped Side Effects
-------------------

The `defer()` primitive generalized `on.exit()`, allowing a function to perform cleanup work after the parent invoking function has finished.

``` r
scope_dir <- function(dir) {
  owd <- setwd(dir)
  defer(setwd(owd), envir = parent.frame())
}
```

The `scope_dir()` function can be called to set the working directory, and restore the working directory when the caller is done. For example:

``` r
in_dir <- function(dir) {
  scope_dir(dir)
  print(basename(getwd()))
}
print(basename(getwd()))
```

    ## [1] "later"

``` r
in_dir("tests")
```

    ## [1] "tests"

``` r
print(basename(getwd()))
```

    ## [1] "later"

Signals
-------

Register a signal handler using `on()`, and fire events with `signal()`. Useful for long-range communication between functions / objects.

``` r
set.seed(1)
make_worker <- function(i) {
  force(i)
  function() {
    signal("work", i)
  }
}

workers <- lapply(seq_len(5), make_worker)
manager <- function() {
  
  counter <- 0
  on("work", function(data) {
    cat(sprintf("Received data: '%s'\n", data), sep = "")
    counter <<- counter + 1
  })
  
  # Run for 2 milliseconds
  time <- Sys.time()
  while (Sys.time() - time < 0.002) {
    worker <- sample(workers, 1)[[1]]
    worker()
  }
  
  cat(sprintf("Executed %s tasks.\n", counter), sep = "")
}

manager()
```

    ## Received data: '2'
    ## Received data: '2'
    ## Received data: '3'
    ## Received data: '5'
    ## Received data: '2'
    ## Received data: '5'
    ## Received data: '5'
    ## Executed 7 tasks.

Inspiration & Credit
--------------------

This package is heavily inspired by Jim Hester's [withr](https://github.com/jimhester/withr) package.

The main 'trick' that enabled the generalization of `on.exit()` was provided in a mailing list post here, by Peter Meilstrup: <https://stat.ethz.ch/pipermail/r-devel/2013-November/067874.html>
