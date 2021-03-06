---
title: "Problem set 9"
linktitle: "Problem set 9"
date: "2021-12-06"
due_date: "2021-12-06"
due_time: "11:59 PM"
menu:
  assignment:
    parent: Problem sets
    weight: 9
type: docs
editor_options: 
  chunk_output_type: console
---

This assignment will give you practice generating synthetic data and building in causal effects.

These two examples will be incredibly helpful:

- [Generating random numbers](/example/random-numbers/)
- [The ultimate guide to generating synthetic data for causal inference](/example/synthetic-data/)


You'll be doing all your R work in R Markdown. You can download a zipped file of a pre-made project here:

- [<i class="fas fa-file-archive"></i> `problem-set-9.zip`](/projects/problem-set-9.zip)

And as always, if you're struggling, *please* talk to me. Work with classmates too (especially for this assignment!). Don't suffer in silence!


## Instructions

1. If you're using R on your own computer, download this file, [*unzip it*](/resource/unzipping/), and double click on the file named `problem-set-9.Rproj`: [<i class="fas fa-file-archive"></i> `problem-set-9.zip`](/projects/problem-set-9.zip)

    You'll need to make sure you have these packages installed on your computer: `tidyverse`, `broom`, `ggdag`, and `scales`. If you try to load one of those packages with `library(tidyverse)` or `library(ggdag)`, etc., and R gives an error that the package is missing, use the "Packages" panel in RStudio to install it.

    (Alternatively, you can open the project named "Problem Set 9" on RStudio.cloud and complete the assignment in your browser without needing to install anything. [This link should take you to the project](https://rstudio.cloud/spaces/160211/project/2762034)—if it doesn't, log in and look for the project named "Problem Set 9.")

2. Rename the R Markdown file named `your-name_problem-set-9.Rmd` to something that matches your name and open it in RStudio.

3. Complete the tasks given in the R Markdown file. You can remove any of the question text if you want.

    You can definitely copy, paste, and adapt from other code in the document or [the example guide](/example/synthetic-data/)—don't try to write everything from scratch!.

    You'll need to insert your own code chunks. Rather than typing them by hand (that's tedious!), use the "Insert" button at the top of the editing window, or press  <kbd>⌥</kbd> + <kbd>⌘</kbd> + <kbd>I</kbd> on macOS, or <kbd>ctrl</kbd> + <kbd>alt</kbd> + <kbd>I</kbd> on Windows.

    <img src="/img/assignments/insert-chunk-button.png" width="19%" />

    Remember that you can run an entire chunk by clicking on the green play arrow in the top right corner of the chunk. You can also run lines of code line-by-line if you place your cursor on some R code and press <kbd>⌘</kbd> + <kbd>enter</kbd> (for macOS users) or <kbd>ctrl</kbd> + <kbd>enter</kbd> (for Windows users).

    Make sure you run each chunk sequentially. If you run a chunk in the middle of the document without running previous ones, it might not work, since previous chunks might do things that later chunks depend on.

4. When you're all done, click on the "Knit" button at the top of the editing window and create an HTML or Word version (or PDF if you've [installed **tinytex**](/resource/install/#install-tinytex)) of your document. Upload that file to iCollege.

<img src="/img/assignments/knit-button.png" width="30%" />
