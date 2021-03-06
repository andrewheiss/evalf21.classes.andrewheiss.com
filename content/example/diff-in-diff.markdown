---
title: "Difference-in-differences"
linktitle: "Diff-in-diff"
date: "2020-03-04"
menu:
  example:
    parent: Examples
    weight: 7
type: docs
toc: true
editor_options: 
  chunk_output_type: console
---
<script src="/rmarkdown-libs/kePrint/kePrint.js"></script>
<link href="/rmarkdown-libs/lightable/lightable.css" rel="stylesheet" />



## Video walk-through

If you want to follow along with this example, you can download the data below:

- [<i class="fas fa-table"></i> `injury.csv`](/data/injury.csv)

There's a set of videos that walks through each section below. To make it easier for you to jump around the video examples, I cut the long video into smaller pieces and included them all in [one YouTube playlist](https://www.youtube.com/playlist?list=PLS6tnpTr39sHw3FevrihLn2Ly8pSCUUag).

- [Getting started](https://www.youtube.com/watch?v=u5iEtpITL3s&list=PLS6tnpTr39sHw3FevrihLn2Ly8pSCUUag)
- [Load and clean data](https://www.youtube.com/watch?v=15BprphMT1g&list=PLS6tnpTr39sHw3FevrihLn2Ly8pSCUUag)
- [Exploratory data analysis](https://www.youtube.com/watch?v=SkPLMBkB06o&list=PLS6tnpTr39sHw3FevrihLn2Ly8pSCUUag)
- [Diff-in-diff manually](https://www.youtube.com/watch?v=56KPL2_nSxQ&list=PLS6tnpTr39sHw3FevrihLn2Ly8pSCUUag)
- [Diff-in-diff with regression](https://www.youtube.com/watch?v=2JctZFGIYWw&list=PLS6tnpTr39sHw3FevrihLn2Ly8pSCUUag)

You can also watch the playlist (and skip around to different sections) here:

<div class="embed-responsive embed-responsive-16by9">
<iframe class="embed-responsive-item" src="https://www.youtube.com/embed/playlist?list=PLS6tnpTr39sHw3FevrihLn2Ly8pSCUUag" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>


## Program background

In 1980, Kentucky raised its cap on weekly earnings that were covered by worker's compensation. We want to know if this new policy caused workers to spend more time unemployed. If benefits are not generous enough, then workers could sue companies for on-the-job injuries, while overly generous benefits could cause moral hazard issues and induce workers to be more reckless on the job, or to claim that off-the-job injuries were incurred while at work.

The main outcome variable we care about is `log_duration` (in the original data as `ldurat`, but we rename it to be more human readable), or the logged duration (in weeks) of worker's compensation benefits. We log it because the variable is fairly skewed---most people are unemployed for a few weeks, with some unemployed for a long time. The policy was designed so that the cap increase did not affect low-earnings workers, but did affect high-earnings workers, so we use low-earnings workers as our control group and high-earnings workers as our treatment group.

The data is included in the **wooldridge** R package as the `injury` dataset, and if you install the package, load it with `library(wooldridge)`, and run `?injury` in the console, you can see complete details about what's in it. To give you more practice with loading data from external files, I exported the injury data as a CSV file (using `write_csv(injury, "injury.csv")`) and included it here.

These are the main columns we'll worry about for now:

- `durat` (which we'll rename to `duration`): Duration of unemployment benefits, measured in weeks
- `ldurat` (which we'll rename to `log_duration`): Logged version of `durat` (`log(durat)`)
- `after_1980` (which we'll rename to `after_1980`): Indicator variable marking if the observation happened before (0) or after (1) the policy change in 1980. This is our **time** (or *before/after* variable)
- `highearn`: Indicator variable marking if the observation is a low (0) or high (1) earner. This is our **group** (or *treatment/control*) variable


## Load and clean data

First, let's download the dataset (if you haven't already), put in a folder named `data`, and load it:

- [<i class="fas fa-table"></i> `injury.csv`](/data/injury.csv)


```r
library(tidyverse)  # ggplot(), %>%, mutate(), and friends
library(broom)  # Convert models to data frames
library(scales)  # Format numbers with functions like comma(), percent(), and dollar()
library(modelsummary)  # Create side-by-side regression tables
```




```r
# Load the data.
# It'd be a good idea to click on the "injury_raw" object in the Environment
# panel in RStudio to see what the data looks like after you load it
injury_raw <- read_csv("data/injury.csv")
```



Next we'll clean up the `injury_raw` data a little to limit the data to just observations from Kentucky. we'll also rename some awkwardly-named columns:


```r
injury <- injury_raw %>%
  filter(ky == 1) %>%
  # The syntax for rename is `new_name = original_name`
  rename(duration = durat, log_duration = ldurat,
         after_1980 = afchnge)
```


## Exploratory data analysis

First we can look at the distribution of unemployment benefits across high and low earners (our control and treatment groups):


```r
ggplot(data = injury, aes(x = duration)) +
  # binwidth = 8 makes each column represent 2 months (8 weeks)
  # boundary = 0 make it so the 0-8 bar starts at 0 and isn't -4 to 4
  geom_histogram(binwidth = 8, color = "white", boundary = 0) +
  facet_wrap(vars(highearn))
```

<img src="/example/diff-in-diff_files/figure-html/duration-histogram-1.png" width="75%" style="display: block; margin: auto;" />

The distribution is really skewed, with most people in both groups getting between 0-8 weeks of benefits (and a handful with more than 180 weeks! that's 3.5 years!)

If we use the log of duration, we can get a less skewed distribution that works better with regression models:


```r
ggplot(data = injury, mapping = aes(x = log_duration)) +
  geom_histogram(binwidth = 0.5, color = "white", boundary = 0) +
  # Uncomment this line if you want to exponentiate the logged values on the
  # x-axis. Instead of showing 1, 2, 3, etc., it'll show e^1, e^2, e^3, etc. and
  # make the labels more human readable
  # scale_x_continuous(labels = trans_format("exp", format = round)) +
  facet_wrap(vars(highearn))
```

<img src="/example/diff-in-diff_files/figure-html/log-duration-histogram-1.png" width="75%" style="display: block; margin: auto;" />

We should also check the distribution of unemployment before and after the policy change. Copy/paste one of the histogram chunks and change the faceting:


```r
ggplot(data = injury, mapping = aes(x = log_duration)) +
  geom_histogram(binwidth = 0.5, color = "white", boundary = 0) +
  facet_wrap(vars(after_1980))
```

<img src="/example/diff-in-diff_files/figure-html/log-duration-before-after-histogram-1.png" width="75%" style="display: block; margin: auto;" />

The distributions look normal-ish, but we can't really easily see anything different between the before/after and treatment/control groups. We can plot the averages, though. There are a few different ways we can do this.

You can use a `stat_summary()` layer to have ggplot calculate summary statistics like averages. Here we just calculate the mean:


```r
ggplot(injury, aes(x = factor(highearn), y = log_duration)) +
  geom_point(size = 0.5, alpha = 0.2) +
  stat_summary(geom = "point", fun = "mean", size = 5, color = "red") +
  facet_wrap(vars(after_1980))
```

<img src="/example/diff-in-diff_files/figure-html/plot-means-with-points-1.png" width="75%" style="display: block; margin: auto;" />

But we can also calculate the mean and 95% confidence interval:


```r
ggplot(injury, aes(x = factor(highearn), y = log_duration)) +
  stat_summary(geom = "pointrange", size = 1, color = "red",
               fun.data = "mean_se", fun.args = list(mult = 1.96)) +
  facet_wrap(vars(after_1980))
```

<img src="/example/diff-in-diff_files/figure-html/plot-means-with-pointrange-1.png" width="75%" style="display: block; margin: auto;" />

We can already start to see the classical diff-in-diff plot! It looks like high earners after 1980 had longer unemployment on average.

We can also use `group_by()` and `summarize()` to figure out group means before sending the data to ggplot. I prefer doing this because it gives me more control over the data that I'm plotting:


```r
plot_data <- injury %>%
  # Make these categories instead of 0/1 numbers so they look nicer in the plot
  mutate(highearn = factor(highearn, labels = c("Low earner", "High earner")),
         after_1980 = factor(after_1980, labels = c("Before 1980", "After 1980"))) %>%
  group_by(highearn, after_1980) %>%
  summarize(mean_duration = mean(log_duration),
            se_duration = sd(log_duration) / sqrt(n()),
            upper = mean_duration + (1.96 * se_duration),
            lower = mean_duration + (-1.96 * se_duration))

ggplot(plot_data, aes(x = highearn, y = mean_duration)) +
  geom_pointrange(aes(ymin = lower, ymax = upper),
                  color = "darkgreen", size = 1) +
  facet_wrap(vars(after_1980))
```

<img src="/example/diff-in-diff_files/figure-html/plot-pointrange-manual-1.png" width="75%" style="display: block; margin: auto;" />

Or, plotted in the more standard diff-in-diff format:


```r
ggplot(plot_data, aes(x = after_1980, y = mean_duration, color = highearn)) +
  geom_pointrange(aes(ymin = lower, ymax = upper), size = 1) +
  # The group = highearn here makes it so the lines go across categories
  geom_line(aes(group = highearn))
```

<img src="/example/diff-in-diff_files/figure-html/plot-pointrange-manual-no-facet-1.png" width="75%" style="display: block; margin: auto;" />


## Diff-in-diff by hand

We can find that exact difference by filling out the 2x2 before/after treatment/control table:

|              | Before 1980 | After 1980 |         ???         |
|--------------|:-----------:|:----------:|:-----------------:|
| Low earners  |      A      |      B     |       B ??? A       |
| High earners |      C      |      D     |       D ??? C       |
| ???            |    C ??? A    |    D ??? B   | (D ??? C) ??? (B ??? A) |


A combination of `group_by()` and `summarize()` makes this really easy:


```r
diffs <- injury %>%
  group_by(after_1980, highearn) %>%
  summarize(mean_duration = mean(log_duration),
            # Calculate average with regular duration too, just for fun
            mean_duration_for_humans = mean(duration))
diffs
## # A tibble: 4 ?? 4
## # Groups:   after_1980 [2]
##   after_1980 highearn mean_duration mean_duration_for_humans
##        <dbl>    <dbl>         <dbl>                    <dbl>
## 1          0        0          1.13                     6.27
## 2          0        1          1.38                    11.2 
## 3          1        0          1.13                     7.04
## 4          1        1          1.58                    12.9
```

We can pull each of these numbers out of the table with some `filter()`s and `pull()`:


```r
before_treatment <- diffs %>%
  filter(after_1980 == 0, highearn == 1) %>%
  pull(mean_duration)

before_control <- diffs %>%
  filter(after_1980 == 0, highearn == 0) %>%
  pull(mean_duration)

after_treatment <- diffs %>%
  filter(after_1980 == 1, highearn == 1) %>%
  pull(mean_duration)

after_control <- diffs %>%
  filter(after_1980 == 1, highearn == 0) %>%
  pull(mean_duration)

diff_treatment_before_after <- after_treatment - before_treatment
diff_treatment_before_after
## [1] 0.2

diff_control_before_after <- after_control - before_control
diff_control_before_after
## [1] 0.0077

diff_diff <- diff_treatment_before_after - diff_control_before_after
diff_diff
## [1] 0.19
```

The diff-in-diff estimate is 0.19, which means that the program causes an increase in unemployment duration of 0.19 logged weeks. Logged weeks is nonsensical, though, so we have to interpret it with percentages ([here's a handy guide!](https://stats.stackexchange.com/a/18639/3025); this is Example B, where the dependent/outcome variable is logged). Receiving the treatment (i.e. being a high earner after the change in policy) causes a 19% increase in the length of unemployment.


```r
ggplot(diffs, aes(x = as.factor(after_1980),
                  y = mean_duration,
                  color = as.factor(highearn))) +
  geom_point() +
  geom_line(aes(group = as.factor(highearn))) +
  # If you use these lines you'll get some extra annotation lines and
  # labels. The annotate() function lets you put stuff on a ggplot that's not
  # part of a dataset. Normally with geom_line, geom_point, etc., you have to
  # plot data that is in columns. With annotate() you can specify your own x and
  # y values.
  annotate(geom = "segment", x = "0", xend = "1",
           y = before_treatment, yend = after_treatment - diff_diff,
           linetype = "dashed", color = "grey50") +
  annotate(geom = "segment", x = "1", xend = "1",
           y = after_treatment, yend = after_treatment - diff_diff,
           linetype = "dotted", color = "blue") +
  annotate(geom = "label", x = "1", y = after_treatment - (diff_diff / 2),
           label = "Program effect", size = 3)
```

<img src="/example/diff-in-diff_files/figure-html/nice-diff-diff-plot-1.png" width="75%" style="display: block; margin: auto;" />

```r

# Here, all the as.factor changes are directly in the ggplot code. I generally
# don't like doing this and prefer to do that separately so there's less typing
# in the ggplot code, like this:
#
# diffs <- diffs %>%
#   mutate(after_1980 = as.factor(after_1980), highearn = as.factor(highearn))
#
# ggplot(diffs, aes(x = after_1980, y = avg_durat, color = highearn)) +
#   geom_line(aes(group = highearn))
```


## Diff-in-diff with regression

Calculating all the pieces by hand like that is tedious, so we can use regression to do it instead! Remember that we need to include indicator variables for treatment/control and for before/after, as well as the interaction of the two. Here's what the math equation looks like:

$$
`\begin{aligned}
\log(\text{duration}) = &\alpha + \beta \ \text{highearn} + \gamma \ \text{after_1980} + \\
& \delta \ (\text{highearn} \times \text{after_1980}) + \epsilon
\end{aligned}`
$$

The `\(\delta\)` coefficient is the effect we care about in the end---that's the diff-in-diff estimator.


```r
model_small <- lm(log_duration ~ highearn + after_1980 + highearn * after_1980,
                  data = injury)
tidy(model_small)
## # A tibble: 4 ?? 5
##   term                estimate std.error statistic   p.value
##   <chr>                  <dbl>     <dbl>     <dbl>     <dbl>
## 1 (Intercept)          1.13       0.0307    36.6   1.62e-263
## 2 highearn             0.256      0.0474     5.41  6.72e-  8
## 3 after_1980           0.00766    0.0447     0.171 8.64e-  1
## 4 highearn:after_1980  0.191      0.0685     2.78  5.42e-  3
```

The coefficient for `highearn:after_1980` is the same as what we found by hand, as it should be! It is statistically significant, so we can be fairly confident that it is not 0.


## Diff-in-diff with regression + controls

One advantage to using regression for diff-in-diff is that we can include control variables to help isolate the effect. For example, perhaps claims made by construction or manufacturing workers tend to have longer duration than claims made workers in other industries. Or maybe those claiming back injuries tend to have longer claims than those claiming head injuries. We might also want to control for worker demographics such as gender, marital status, and age.

Let's estimate an expanded version of the basic regression model with the following additional variables:

- `male`
- `married`
- `age`
- `hosp` (1 = hospitalized)
- `indust` (1 = manuf, 2 = construc, 3 = other)
- `injtype` (1-8; categories for different types of injury)
- `lprewage` (log of wage prior to filing a claim)

*Important*: `indust` and `injtype` are in the dataset as numbers (1-3 and 1-8), but they're actually categories. We have to tell R to treat them as categories (or factors), otherwise it'll assume that you can have an injury type of 3.46 or something impossible.


```r
# Convert industry and injury type to categories/factors
injury_fixed <- injury %>%
  mutate(indust = as.factor(indust),
         injtype = as.factor(injtype))
```


```r
model_big <- lm(log_duration ~ highearn + after_1980 + highearn * after_1980 +
                  male + married + age + hosp + indust + injtype + lprewage,
                data = injury_fixed)
tidy(model_big)
## # A tibble: 18 ?? 5
##    term                estimate std.error statistic   p.value
##    <chr>                  <dbl>     <dbl>     <dbl>     <dbl>
##  1 (Intercept)         -1.53      0.422       -3.62 2.98e-  4
##  2 highearn            -0.152     0.0891      -1.70 8.86e-  2
##  3 after_1980           0.0495    0.0413       1.20 2.31e-  1
##  4 male                -0.0843    0.0423      -1.99 4.64e-  2
##  5 married              0.0567    0.0373       1.52 1.29e-  1
##  6 age                  0.00651   0.00134      4.86 1.19e-  6
##  7 hosp                 1.13      0.0370      30.5  5.20e-189
##  8 indust2              0.184     0.0541       3.40 6.87e-  4
##  9 indust3              0.163     0.0379       4.32 1.60e-  5
## 10 injtype2             0.935     0.144        6.51 8.29e- 11
## 11 injtype3             0.635     0.0854       7.44 1.19e- 13
## 12 injtype4             0.555     0.0928       5.97 2.49e-  9
## 13 injtype5             0.641     0.0854       7.51 7.15e- 14
## 14 injtype6             0.615     0.0863       7.13 1.17e- 12
## 15 injtype7             0.991     0.191        5.20 2.03e-  7
## 16 injtype8             0.434     0.119        3.65 2.64e-  4
## 17 lprewage             0.284     0.0801       3.55 3.83e-  4
## 18 highearn:after_1980  0.169     0.0640       2.64 8.38e-  3

# Extract just the diff-in-diff estimate
diff_diff_controls <- tidy(model_big) %>%
  filter(term == "highearn:after_1980") %>%
  pull(estimate)
```

After controlling for a host of demographic controls, the diff-in-diff estimate is smaller (0.17), indicating that the policy caused a 17% increase in the duration of weeks unemployed following a workplace injury. It is smaller because the other independent variables now explain some of the variation in `log_duration`.


## Comparison of results

We can put the model coefficients side-by-side to compare the value for `highearn:after_1980` as we change the model.


```r
modelsummary(list("Simple" = model_small, "Full" = model_big))
```

<table style="NAborder-bottom: 0; width: auto !important; margin-left: auto; margin-right: auto;" class="table">
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:center;"> Simple </th>
   <th style="text-align:center;"> Full </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> (Intercept) </td>
   <td style="text-align:center;"> 1.126*** </td>
   <td style="text-align:center;"> -1.528*** </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;"> (0.031) </td>
   <td style="text-align:center;"> (0.422) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> highearn </td>
   <td style="text-align:center;"> 0.256*** </td>
   <td style="text-align:center;"> -0.152+ </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;"> (0.047) </td>
   <td style="text-align:center;"> (0.089) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> after_1980 </td>
   <td style="text-align:center;"> 0.008 </td>
   <td style="text-align:center;"> 0.050 </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;"> (0.045) </td>
   <td style="text-align:center;"> (0.041) </td>
  </tr>
  <tr>
   <td style="text-align:left;background-color: #F6D645 !important;"> highearn ?? after_1980 </td>
   <td style="text-align:center;background-color: #F6D645 !important;"> 0.191** </td>
   <td style="text-align:center;background-color: #F6D645 !important;"> 0.169** </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;"> (0.069) </td>
   <td style="text-align:center;"> (0.064) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> male </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> -0.084* </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> (0.042) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> married </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> 0.057 </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> (0.037) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> age </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> 0.007*** </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> (0.001) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hosp </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> 1.130*** </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> (0.037) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> indust2 </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> 0.184*** </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> (0.054) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> indust3 </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> 0.163*** </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> (0.038) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> injtype2 </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> 0.935*** </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> (0.144) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> injtype3 </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> 0.635*** </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> (0.085) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> injtype4 </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> 0.555*** </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> (0.093) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> injtype5 </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> 0.641*** </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> (0.085) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> injtype6 </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> 0.615*** </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> (0.086) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> injtype7 </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> 0.991*** </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> (0.191) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> injtype8 </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> 0.434*** </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> (0.119) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lprewage </td>
   <td style="text-align:center;">  </td>
   <td style="text-align:center;"> 0.284*** </td>
  </tr>
  <tr>
   <td style="text-align:left;box-shadow: 0px 1pxborder-bottom: 1px solid">  </td>
   <td style="text-align:center;box-shadow: 0px 1pxborder-bottom: 1px solid">  </td>
   <td style="text-align:center;box-shadow: 0px 1pxborder-bottom: 1px solid"> (0.080) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Num.Obs. </td>
   <td style="text-align:center;"> 5626 </td>
   <td style="text-align:center;"> 5347 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> R2 </td>
   <td style="text-align:center;"> 0.021 </td>
   <td style="text-align:center;"> 0.190 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> R2 Adj. </td>
   <td style="text-align:center;"> 0.020 </td>
   <td style="text-align:center;"> 0.187 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> AIC </td>
   <td style="text-align:center;"> 18654.0 </td>
   <td style="text-align:center;"> 16684.8 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> BIC </td>
   <td style="text-align:center;"> 18687.2 </td>
   <td style="text-align:center;"> 16809.9 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Log.Lik. </td>
   <td style="text-align:center;"> -9321.997 </td>
   <td style="text-align:center;"> -8323.388 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> F </td>
   <td style="text-align:center;"> 39.540 </td>
   <td style="text-align:center;"> 73.459 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Deviance </td>
   <td style="text-align:center;"> 9055.93 </td>
   <td style="text-align:center;"> 7042.41 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> DF Resid </td>
   <td style="text-align:center;"> 5622 </td>
   <td style="text-align:center;"> 5329 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sigma </td>
   <td style="text-align:center;"> 1.269 </td>
   <td style="text-align:center;"> 1.150 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Statistics </td>
   <td style="text-align:center;"> 39.540 </td>
   <td style="text-align:center;"> 73.459 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> p </td>
   <td style="text-align:center;"> 0.000 </td>
   <td style="text-align:center;"> 0.000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> DF </td>
   <td style="text-align:center;"> 3 </td>
   <td style="text-align:center;"> 17 </td>
  </tr>
</tbody>
<tfoot><tr><td style="padding: 0; " colspan="100%">
<sup></sup> + p &lt; 0.1, * p &lt; 0.05, ** p &lt; 0.01, *** p &lt; 0.001</td></tr></tfoot>
</table>
