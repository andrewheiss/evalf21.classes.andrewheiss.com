---
title: "Problem set 4"
linktitle: "Problem set 4"
date: "2021-10-18"
due_date: "2021-10-18"
due_time: "11:59 PM"
menu:
  assignment:
    parent: Problem sets
    weight: 4
type: docs
editor_options: 
  chunk_output_type: console
---

{{% div note %}}
**IMPORTANT**: This looks like a lot of work, but again, it's mostly copying/pasting chunks of code and changing things. 
{{% /div %}}

For this problem set, you'll practice running difference-in-differences analysis with R, both manually and with regression. This example will be incredibly useful for you:

- [Difference-in-differences](/example/diff-in-diff/)

You'll be doing all your R work in R Markdown. You can download a zipped file of a pre-made project here:

- [<i class="fas fa-file-archive"></i> `problem-set-4.zip`](/projects/problem-set-4.zip)

And as always, if you're struggling, *please* talk to me. Work with classmates too (especially for this assignment!). Don't suffer in silence!


## Instructions

1. If you're using R on your own computer, download this file, [*unzip it*](/resource/unzipping/), and double click on the file named `problem-set-4.Rproj`: [<i class="fas fa-file-archive"></i> `problem-set-4.zip`](/projects/problem-set-4.zip)

    You'll need to make sure you have these packages installed on your computer: `tidyverse`, `haven`, and `broom`. If you try to load one of those packages with `library(tidyverse)` or `library(haven)`, etc., and R gives an error that the package is missing, use the "Packages" panel in RStudio to install it.

    (Alternatively, you can open the project named "Problem Set 4" on RStudio.cloud and complete the assignment in your browser without needing to install anything. If you don't have access to the class RStudio.cloud account, *please let me know as soon as possible*. [This link should take you to the project](https://rstudio.cloud/spaces/160211/project/2762021)—if it doesn't, log in and look for the project named "Problem Set 4.")

2. Rename the R Markdown file named `your-name_problem-set-4.Rmd` to something that matches your name and open it in RStudio.

3. Complete the tasks given in the R Markdown file. There are questions **marked in bold** (e.g. `**What is the ATE?**`). Your job is to answer those questions. You don't need to put your answers in bold, and you can remove the question text if you want.

    Fill out code in the empty chunks provided (you can definitely copy, paste, and adapt from other code in the document or [the example page on diff-in-diff](/example/diff-in-diff/)—don't try to write everything from scratch!).

    You'll need to insert your own code chunks. Rather than typing them by hand (that's tedious!), use the "Insert" button at the top of the editing window, or press  <kbd>⌥</kbd> + <kbd>⌘</kbd> + <kbd>I</kbd> on macOS, or <kbd>ctrl</kbd> + <kbd>alt</kbd> + <kbd>I</kbd> on Windows.

    <img src="/img/assignments/insert-chunk-button.png" width="19%" />

    Remember that you can run an entire chunk by clicking on the green play arrow in the top right corner of the chunk. You can also run lines of code line-by-line if you place your cursor on some R code and press <kbd>⌘</kbd> + <kbd>enter</kbd> (for macOS users) or <kbd>ctrl</kbd> + <kbd>enter</kbd> (for Windows users).

    Make sure you run each chunk sequentially. If you run a chunk in the middle of the document without running previous ones, it might not work, since previous chunks might do things that later chunks depend on.

4. When you're all done, click on the "Knit" button at the top of the editing window and create an HTML or Word version (or PDF if you've [installed **tinytex**](/resource/install/#install-tinytex)) of your document. Upload that file to iCollege.

<img src="/img/assignments/knit-button.png" width="30%" />
