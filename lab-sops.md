---
title: "SOPs for The Lab @ DC"
author: "Ryan T. Moore"
date: "2022-09-20"
site: bookdown::bookdown_site
documentclass: book
bibliography: [book.bib]
biblio-style: apalike
link-citations: yes
github-repo: thelabdc/LAB-SOP-experiments
url: 'https\://thelab.dc.gov/LAB-SOP-experiments/'
description: "Standard Operating Procedures for Experiments at The Lab @ DC."
code_folding: show
---

# Preface {.unnumbered}

Placeholder


## Colophon {.unnumbered}

<!--chapter:end:index.Rmd-->

# Getting Started

Before the design of the experiment

- [ ] Copy the Asana project template for your project
- [ ] Set up GitHub repository for the study
- [ ] Set up Google Drive folders for the study
- [ ] See the Asana project template for more

The pre-analysis plan template lives at 

<!--chapter:end:01-Introduction.Rmd-->

# Design

## Units

We carefully define the units of observation. They maybe households, automobile
registrations, 911 callers, or students, about which we are measuring
application status, traffic tickets, primary care visits, or days of attendance
in school, respectively.

## Level of Randomization

We define the level of randomization, which is not always the same as
the unit of observation. In particular, the units of observation may be assigned
in _clusters_. For example, we may assign some _classrooms_ to a particular
teacher-based intervention, and all students in intervention classrooms are
assigned to the same intervention. When clusters are meaningful, this has
substantial implications for our design and analysis. 

Generally, we prefer to randomize at lower levels of aggregation -- at the
student level rather than the classroom level, e.g. -- because we will have more
randomization units. However, there are often logistical or statistical reasons
for assigning conditions in clusters. For example, logistically, a teacher can
only deliver one particular curriculum to their class; statistically, we may be
concerned about interference between students, where students in one condition
interact with students in the other, and causal effects of the intervention are
difficult to isolate.

## Blocking on Background Characteristics

In order to create balance on potential outcomes, which promotes less estimation error and more precision, we block using prognostic covariates. A _blocked_ randomization first creates groups of similar randomization units, and then randomizes within those groups. In a _random allocation_, by contrast, one assigns a proportion of units to each treatment condition from a single pool. See @moore12 for discussion.

### Examples

The Lab's TANF recertification experiment [@mooganmin22] blocked participants on
service center and assigned visit date.

## Setting the Assignment Seed {#sec-set-seed}

Whenever our design or analysis involves randomization, we set the seed so that
our results can be perfectly replicated.^[We randomize treatment assignment in
experiments, and we often simulate in power analysis and randomization
inference.]

To set the random seed, run at the R prompt


```r
sample(1e9, 1)
```

then copy and paste the result as the argument of `set.seed()`. If the result of
`sample(1e9, 1)` is `SSSSSSSSS`, then put


```r
set.seed(SSSSSSSSS)
```

at the top of the `.R` file, just after the `library()` commands that load and
attach that file's packages. In a short random assignment file, e.g., we might
have


```r
# Packages:
library(dplyr) 

# Set seed:
set.seed(SSSSSSSSS)

# Conduct Bernoulli randomization:
df <- df %>% mutate(
  treatment = sample(0:1, 
                     size = n(),
                     replace = TRUE))
```


## Power

We conduct power analysis to determine how precise our experiment are likely to
be. Power analysis should go beyond sample size, and attempt to account for as
many features of the design, implementation, and analysis as possible. Sometimes
we can achieve this with "formula-based" power strategies; other times we need
to use simulation-based techniques. Power analyses should be conducted in code,
so that they are replicable. If we use an online calculator for a quick
calculation, we replicate the results in code.^[An online power calculator is
available from [EGAP](https://egap.shinyapps.io/Power_Calculator/), e.g.]

If our design and analysis plan match the assumptions of a formula-based power
calculation well, we perform a formula-based power calculation. For example, if
the design is complete randomization and the analysis plan is to conduct a
two-sample $t$-test, we might use R's `power.t.test()`, as below. However, if
the design includes blocked assignment in several waves, untreated units stay in
the pool across waves, and assignment probabilities vary by wave, with an
analysis plan of covariance-adjusted Lin [estimation](#sec-lin) with strong
covariates, then we need to use simulation. If we can't find a formula-based
approach that sufficiently approximates our design and analysis plan, we use
simulation.



### Formula-based Power Analysis

An example of formula-based power calculation, where the design is complete randomization and the analysis plans for a two-sample $t$-test:


```{.r .fold.hide}
power_out <- power.t.test(delta = 1, 
                          sd = 1,
                          sig.level = 0.05,
                          power = 0.8)
power_out
```

```
## 
##      Two-sample t test power calculation 
## 
##               n = 16.71477
##           delta = 1
##              sd = 1
##       sig.level = 0.05
##           power = 0.8
##     alternative = two.sided
## 
## NOTE: n is number in *each* group
```

For two equally-sized samples drawn from a population with standard normal outcomes, we need 17 observations in each group to have a probability of 0.8 of detecting a true effect that is one standard deviation of the outcome in size, where "detecting" means rejecting a null hypothesis of $H_0: \mu_{Tr} = \mu_{Co}$ against an alternative of $H_a: \mu_{Tr} \neq \mu_{Co}$ using $\alpha = 0.05$.

### Simulation-based Power Analysis

Simulation-based power analysis allows us to estimate the power of any
combination of randomization technique, missing data treatment, estimation
strategy, etc. that we like.

Create some data to illustrate simulation-based power analysis:

```r
library(estimatr)
library(here)
library(tidyverse)

set.seed(988869862)

n_samp <- 100
df <- tibble(x = rchisq(n_samp, df = 3),
             z = rbinom(n_samp, 1, prob = 0.5),
             y = x + z + rnorm(n_samp, sd = 1.1))

save(df, file = here("data", "02-01-df.RData"))
```

Suppose the estimation strategy is linear regression $y_i = \beta_0 + \beta_1 z_i +  \beta_2 x_i + \epsilon_i$ with heteroskedasticity-robust HC2 standard errors, and the coefficient of interest is $\beta_1$. Perform 1000 reassignments and determine what proportion of them reveal $\hat{\beta}_1$ that is statistically significant at $\alpha = 0.05$.


```r
n_sims <- 1000
alpha <- 0.05
true_te <- 1

is_stat_sig <- vector("logical", n_sims) # Storage

for(idx in 1:n_sims){
  
  # Re-assign treatment and recalculate outcomes n_sims times:
  # (Note: conditions on original x in df, but not original y.)
  df <- df %>% mutate(z_tmp = rbinom(n_samp, 1, prob = 0.5),
                      y_tmp = true_te * z_tmp + x + rnorm(100, sd = 1.1))
  
  # Estimation:
  lm_out <- lm_robust(y_tmp ~ z_tmp + x, data = df)
  
  # Store p-value:
  stat_sig_tmp <- lm_out$p.value["z_tmp"]
  
  # Store whether true effect is 'detected':
  is_stat_sig[idx] <- (stat_sig_tmp <= alpha)
}

mean(is_stat_sig)
```

```
## [1] 0.995
```

So the probability of detecting the true average treatment effect of 1 is about 0.995. This is high power comes largely from the strongly predictive nature of the covariate `x`. Note that a na??ve formula-based approach that ignores the data generating process estimates the power to be roughly 0.45.

## Balance Checking

To increase our certainty that our treatment and control conditions are balanced on predictive covariates, we compare the distributions of covariates. For example, in @mooganmin22, we describe 

> the median absolute deviation (MAD) of appointment dates is about 0.15 days (about 3.5 hours) or less in 99% of the trios.  In other words, the medians of the treatment and control groups tend to vary by much less than a day across the months and Service Centers.


<!--chapter:end:02-Design.Rmd-->

# Implementation

Here.

<!--chapter:end:03-Implementation.Rmd-->


# Analysis

Placeholder


## Notation
## Estimands
## The Unadjusted ITT {#sec-unadj-itt}
## Adjusting for covariates {#sec-lin}
## Clusters, weights, fixed effects
### Weights
## Randomization Inference {#sec-rand-inf}
## Addressing non-compliance
## Missing Data
## Multiple Comparisons
### Holm-Bonferroni adjustment
### Westfall-Young adjustment
## Posterior Probabilities

<!--chapter:end:04-Analysis.Rmd-->

# Publication

- Public version of GitHub repository
- Academic papers

<!--chapter:end:05-Publication.Rmd-->


# (APPENDIX) Appendix {-} 

Placeholder


## Get R and RStudio
## Read in data files
### `.csv`
### `.xlsx`
## Pipes
## Rename variables
## Create a New Variable
### Create a "wave ID"
### Create a primary key
### Transform an existing variable
### Create a sum of from subset of variables
## Recode a variable's values
## Treat dates as dates
### Make an age variable 
### Make a month-count variable
## Create a completely randomised indicator {#sec-create-treatment}
## Plot a variable
## Calculate a data summary
### Proportions for a categorical variable
### Proportions for binary variables
## Merge (join) dataframes

<!--chapter:end:90-R.Rmd-->


# Managing Code

Placeholder


## Best Practices
## Style
## Projects {#sec-projects}
## Working Directory and Relative Paths {#sec-working-dir}
### See the working directory
### See the project directory
### Create a path with `here()`
## What Packages are Installed?

<!--chapter:end:95-Code.Rmd-->

# Getting Started with `git` and GitHub

We use the `git` version control system and the GitHub web interface to manage
version control.

To get started, read and take steps 1 through 5 in section 18.2, "Initial set
up", [here](https://r-pkgs.org/git.html). (There is nothing R-specific about
these steps; RStudio provides an interface to get SSH set up so that you don't
have to enter your password every time you make a commit or fetch remote code.)

At The Lab, we

* do not store sensitive data on GitHub.

We have a few template repositories:

* The data science team's current [template](https://github.com/thelabdc/dsProjectTemplate)
* A simple, not-organized-by-project-phase one [here](https://github.com/thelabdc/Template)
* An older (out of date?) [one](https://github.com/thelabdc/LAB-PythonProjectTemplate)

<!--chapter:end:97-Git.Rmd-->

# References {-}

<!--chapter:end:99-References.Rmd-->

