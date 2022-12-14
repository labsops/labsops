# Managing Code

## Best Practices

1. Use "R projects" and the `here` package to manage the working directory and
paths. See below in \@ref(sec-projects) and \@ref(sec-working-dir) for some details.
This is how we promote replication and collaboration (including with our future
selves). 

2. Use high-quality names for objects, files, and directories, like `df_resume`, `03-analysis.R`, and `/code/` (not `finalTHING` or `more stuff.R` or `/Next Try/`). Names should be human- and machine-readable. File names should 
    * start with numbers, so that default ordering is correct, as in `00-prelims.R`, `01-power-calcs.R`, etc., 
    * be informative,
    * not have spaces, and
    * separate words with `-` or `_` to enable human reading and globbing. 

Object and directory names also should be informative, not have spaces, and use `_` to
separate words. See these
[slides](https://www2.stat.duke.edu/~rcs46/lectures_2015/01-markdown-git/slides/naming-slides/naming-slides.pdf)
for more detail and painful counterexamples.

3. Use [GitHub](97-Git.Rmd). But do not put sensitive data there.

4. Begin your code file with a `library()` command for every package required
for that file to run. Keep these in alphabetical order.


```r
library(here)
library(tidyverse)
```

5. Do **not** include your "workflow" in your code files. Anything that's
interactive or changes the environment should be excluded. Do not include
`install.packages()`, `View()`, `setwd()`, in your `.R` file. Do not start your `.R` file with `rm(list = ls())`; it is both too strong and too weak.^[It is "too weak" in that you should start a new R session regularly to ensure replicability. Removing objects from the workspace does nothing to packages, session options, graphical `par()`'s, etc.]

6. Don't open or change the original data. Read it in programmatically with,
e.g., `read_csv()` or `read_excel()`. See
[here](https://thelab.dc.gov/LAB-SOP-experiments/r-basics.html#read-in-data-files)
for examples in R.

7. Use the assignment arrow for assignment. The structure: 


```r
new_obj_name <- value_of_that_object
```

8. Use space around operators (write `3 + 4 = 7`, not `3+4=7`) and after a
comma, as in English (write `f(x, y)`, not `f(x,y)`).




## Style

When we write in R, we tend to prefer the `tidyverse` style. See https://style.tidyverse.org for the full treatment with many examples. Of course, there is a package, `styler`, with functions that will style your code for you. See [here](https://r-pkgs.org/r.html?q=style#code-style) for more detail.


## Projects {#sec-projects}

Create a `.Rproj` file in your project's top-level directory, making your
project an "R project". In RStudio, File - New Project - Existing Directory (or
New Directory, if no project folder exists).

Then, to start work, always open the `.Rproj` file. This ensures that you have a fresh
instance of R and RStudio, and the working directory is always the same. The working directory is the top-level directory (that is, the directory within which the `.Rproj` file lives).


## Working Directory and Relative Paths {#sec-working-dir}

The "working directory" is the directory where R will look for data, code files,
etc. and save your output objects (a new `.csv` or a plot `.pdf`) by default.

Opening the `.Rproj` file ensures that the working directory always starts at
the top-level directory of your project.

We create paths to our objects using the `here` package. This ensures that a) we
are not hard-coding a path that no one else has (like `~/Me/My
Docs/my_special_folder/my_subfolder/`, etc.), and b) our code is
platform-independent. For more on the `here` package, see
[here](https://github.com/jennybc/here_here).

The code below requires the following packages to be loaded and attached:


```r
library(here)
```

### See the working directory

To see the current working directory, type `getwd()`.

### See the project directory

The "project directory" is the top-level directory of your project. It should be the working directory, as well, if you follow the advice at \@ref(sec-projects), and start work by opening the `.Rproj` file. To see the project directory: 


```r
here()
```

```
## [1] "/Users/ryan.moore/Documents/github/LAB-SOP-experiments"
```

### Create a path with `here()`

I have an object called `02-01-df.RData` in a subdirectory called `/data/`. The
`dir()` function shows what is in that subdirectory:


```r
dir("data")
```

```
## [1] "02-01-df.RData"
```

To see its full path, 


```r
here("data", "02-01-df.RData")
```

```
## [1] "/Users/ryan.moore/Documents/github/LAB-SOP-experiments/data/02-01-df.RData"
```

To use that path to read in the data, I first create the path, then use it to
read in the object:


```r
my_rdata_path <- here("data", "02-01-df.RData")

# Use load() for an .RData object:
load(my_rdata_path) 
```

(These could be combined into a single line.)

The object `02-01-df.RData` contains a single dataframe called `df`. I can now see that dataframe in my environment:


```r
ls()
```

```
## [1] "df"            "my_rdata_path"
```

and examine it


```r
head(df)
```

```
##           x z         y
## 1 5.8873201 0 5.6125511
## 2 2.3428286 0 2.0830483
## 3 3.9439262 0 2.5861546
## 4 3.3373187 1 2.6527890
## 5 2.7946122 0 0.6784908
## 6 0.6943772 0 2.8216648
```

## What Packages are Installed?

To see what packages are installed, 


```r
library()
```

or 


```r
lapply(.libPaths(), dir)
```

```
## [[1]]
##   [1] "abind"         "askpass"       "assertthat"    "backports"    
##   [5] "base"          "base64enc"     "bit"           "bit64"        
##   [9] "blob"          "bookdown"      "boot"          "brew"         
##  [13] "brio"          "broom"         "bslib"         "cachem"       
##  [17] "callr"         "car"           "carData"       "cellranger"   
##  [21] "class"         "cli"           "clipr"         "cluster"      
##  [25] "codetools"     "colorspace"    "commonmark"    "compiler"     
##  [29] "cpp11"         "crayon"        "credentials"   "crosstalk"    
##  [33] "curl"          "data.table"    "datasets"      "DBI"          
##  [37] "dbplyr"        "DeclareDesign" "desc"          "devtools"     
##  [41] "diffobj"       "digest"        "downlit"       "dplyr"        
##  [45] "DT"            "dtplyr"        "ellipsis"      "estimatr"     
##  [49] "evaluate"      "fabricatr"     "fansi"         "farver"       
##  [53] "fastmap"       "fontawesome"   "forcats"       "foreign"      
##  [57] "Formula"       "fs"            "gargle"        "generics"     
##  [61] "gert"          "ggplot2"       "gh"            "gitcreds"     
##  [65] "glue"          "googledrive"   "googlesheets4" "graphics"     
##  [69] "grDevices"     "grid"          "gtable"        "haven"        
##  [73] "here"          "highr"         "hms"           "htmltools"    
##  [77] "htmlwidgets"   "httpuv"        "httr"          "ids"          
##  [81] "ini"           "isoband"       "jquerylib"     "jsonlite"     
##  [85] "KernSmooth"    "knitr"         "labeling"      "later"        
##  [89] "lattice"       "lazyeval"      "lifecycle"     "lme4"         
##  [93] "lmtest"        "lubridate"     "magrittr"      "maptools"     
##  [97] "MASS"          "Matrix"        "MatrixModels"  "memoise"      
## [101] "methods"       "mgcv"          "mime"          "miniUI"       
## [105] "minqa"         "modelr"        "munsell"       "nlme"         
## [109] "nloptr"        "nnet"          "numDeriv"      "openssl"      
## [113] "parallel"      "pbkrtest"      "pillar"        "pkgbuild"     
## [117] "pkgconfig"     "pkgdown"       "pkgload"       "praise"       
## [121] "prettyunits"   "processx"      "profvis"       "progress"     
## [125] "promises"      "ps"            "purrr"         "quantreg"     
## [129] "R6"            "ragg"          "randomizr"     "rappdirs"     
## [133] "rcmdcheck"     "RColorBrewer"  "Rcpp"          "RcppEigen"    
## [137] "readr"         "readxl"        "rematch"       "rematch2"     
## [141] "remotes"       "renv"          "reprex"        "rlang"        
## [145] "rmarkdown"     "roxygen2"      "rpart"         "rprojroot"    
## [149] "rstudioapi"    "rversions"     "rvest"         "sass"         
## [153] "scales"        "selectr"       "sessioninfo"   "shiny"        
## [157] "sourcetools"   "sp"            "SparseM"       "spatial"      
## [161] "splines"       "stats"         "stats4"        "stringi"      
## [165] "stringr"       "survival"      "sys"           "systemfonts"  
## [169] "tcltk"         "testthat"      "textshaping"   "tibble"       
## [173] "tidyr"         "tidyselect"    "tidyverse"     "tinytex"      
## [177] "tools"         "translations"  "tzdb"          "urlchecker"   
## [181] "usethis"       "utf8"          "utils"         "uuid"         
## [185] "vctrs"         "viridisLite"   "vroom"         "waldo"        
## [189] "whisker"       "withr"         "xfun"          "xml2"         
## [193] "xopen"         "xtable"        "yaml"          "zip"          
## [197] "zoo"
```



