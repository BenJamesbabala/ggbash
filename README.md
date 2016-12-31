<!-- README.md is generated from README.Rmd. Please edit that file -->
ggbash: A Faster Way to Write ggplot2
=====================================

[![Travis-CI Build Status](https://travis-ci.org/caprice-j/ggbash.svg?branch=master)](https://travis-ci.org/caprice-j/ggbash) [![Build status](https://ci.appveyor.com/api/projects/status/vfia7i1hfowhpqhs?svg=true)](https://ci.appveyor.com/project/caprice-j/ggbash) [![codecov](https://codecov.io/gh/caprice-j/ggbash/branch/master/graph/badge.svg)](https://codecov.io/gh/caprice-j/ggbash) <!-- [![Coverage Status](https://coveralls.io/repos/github/caprice-j/ggbash/badge.svg)](https://coveralls.io/github/caprice-j/ggbash) --> [![Issue Count](https://codeclimate.com/github/caprice-j/ggbash/badges/issue_count.svg)](https://codeclimate.com/github/caprice-j/ggbash/issues)

ggbash provides a bash-like REPL environment for [ggplot2](https://github.com/tidyverse/ggplot2).

Installation
------------

``` r
devtools::install_github("caprice-j/ggbash")
```

Usage
-----

``` bash
library(ggbash)
ggbash(iris) # start a ggbash session (using iris as main dataset)
```

``` bash
user@host currentDir (iris) $ p Sepal.W Sepal.L c=Sp si=Petal.W
```

``` r
executed:
    ggplot(iris) +
    geom_point(aes(Sepal.Width,
                   Sepal.Length,
                   color=Species,
                   size=Petal.Width))
```

![](README_files/figure-markdown_github/unnamed-chunk-6-1.png)

Or if you just need one figure,

``` r
executed <- drawgg(iris, split_by_space('p Sepal.W Sepal.L c=Sp siz=Petal.W'))
copy_to_clipboard(executed$cmd)
# copied: ggplot(iris) + geom_point(aes(x=Sepal.Width, y=Sepal.Length, colour=Species, size=Petal.Width))
```

Features
--------

### 1. Column Index Match

``` bash
# 'ls' checks column indices
user@host currentDir (iris) $ ls

    1: Sepal.L  (ength)
    2: Sepal.W  (idth)
    3: Petal.L  (ength)
    4: Petal.W  (idth)      
    5: Spec     (ies)

# the same as the above 'p Sepal.W Sepal.L c=Sp si=Petal.W'
user@host currentDir (iris) $ p 2 1 c=5 si=4

# you can mix both notations
user@host currentDir (iris) $ p 2 1 c=5 si=Petal.W
```

### 2. Partial Match

In the first example (`p Sepal.W Sepal.L c=Sp si=Petal.W`), ggbash performs partial matches seven times.

-   **geom names**
    -   `p` matches `geom_point`.
-   **column names**
    -   `Sepal.W` matches `iris$Sepal.Width`.

    -   `Sepal.L` matches `iris$Sepal.Length`.

    -   `Sp` matches `iris$Species`.

    -   `Petal.W` matches `iris$Petal.Width`.

-   **aesthetics names**
    -   `c` matches `color`, which is the aesthetic of geom\_point.
    -   `si` matches `size` ('s' is ambiguous within 'shape', 'size', and 'stroke').

### 3. Implicit Precedence

If unique identification is not possible, `ggbash` tries to guess what is specified instead of returning an error, hoping to achieve least expected keystrokes.

For example, for the input of `p Sepal.W Sepal.L c=Sp s=Petal.W`, `p` ambiguously matched four different geoms, `geom_point`, `geom_path`, `geom_polygon`, and `geom_pointrange`.
`ggbash` determines which geom to use by the predefined order of precedence. `point` is selected in this case.

Similarly, `s` matches three aesthetics, `size`, `shape`, and `stroke`, preferred by this order. Thus, `s` is interpreted as `size` aesthetic.

While it's possible to define your own precedence order through `define_constant_list()`, adding one or two characters might be faster in most cases.

### 4. Piping to copy/save results

``` r
    user@host currentDir (iris) $ cd imageDir

    user@host imageDir (iris) $ p 2 1 c=5 si=4 | png big
    saved as 'iris-Sepal.W-Sepal.L-Sp.png' (big: 1960 x 1440)
    
    user@host imageDir (iris) $ for (i in 2:5) p 1 i | pdf 'iris-for'
    saved as 'iris-for.pdf' (default: 960 x 960)
    
    user@host imageDir (iris) $ p 1 2 col=Sp siz=4 | copy
    copied to clipboard:
    ggplot(iris) + geom_point(aes(x=Sepal.Length,
                                  y=Sepal.Width,
                                  colour=Species,
                                  size=Petal.Width))
```

Goals
-----

ggbash has two main goals:

1.  Provide blazingly fast way to do Exploratory Data Analysis.

    -   less typing by partial and fuzzy match

    -   save plots with auto-generated file names

2.  Make it less stressful to finalize your plots.

    -   adjust colors

    -   rotate axis labels

    -   decide tick label intervals

    -   generate line-wrapped titles
