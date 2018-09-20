
<!--
IMAGES:
Insert them with: ![alt text](image.png)
You can also resize them if needed: convert image.png -resize 50% image.png
If you want to center the image, go through HTML code:
<div style="text-align:center"><img src ="image.png"/></div>

REFERENCES:
For references: Put all the bibTeX references in the file "references.bib"
in the current folder and cite the references as @key or [@key] in the text.
Uncomment the bibliography field in the above header and put a "References"
title wherever you want to display the reference list.
-->
Preambule
---------

The census data are 2 .sav files in the `raw_data/census` folder of the DropBox Ecomore2 folder: `PHC2015_Household_Record.sav` and `PHC2015_Person_Record_Province1.sav`. The first one contains data by household whereas the second one contains data per person. Here we summarise both data set per village and the output is put on the `cleaned_data/census` folder of the DropBox Ecomore2 folder: `hh_by_village.csv` and `prs_by_village.csv` respectively.

Packages
--------

Packages currently installed on the system:

``` r
> installed_packages <- rownames(installed.packages())
```

Packages that we need from [CRAN](https://cran.r-project.org):

``` r
> cran <- c("haven",    # importing .sav files
+           "labelled", # manipulating labelled data
+           "dplyr",    # manipulating data frames
+           "purrr"     # functional programming tools
+           )
```

Installing these packages when not already installed:

``` r
> to_install <- !cran %in% installed_packages
> if (any(to_install)) install.packages(cran[to_install])
```

Loading the packages for interactive use at the command line:

``` r
> invisible(lapply(cran, library, character.only = TRUE))
```

Utilitary functions
-------------------

The function below reads .sav file and converts the output to a proper R data frame:

``` r
> read_sav_data <- function(file) {
+   require(haven)    # read_sav
+   require(labelled) # remove_attributes
+   require(magrittr) # %>% 
+   data <- read_sav(file)
+   data <- to_factor(data)
+   data$EXPORTFILE <- NULL
+   data$FIELDFILE <- NULL
+   names(data) <- data %>%
+     sapply(function(x) attr(x, "label")) %>%
+     unlist() %>%
+     unname() %>%
+     gsub(" +", "_", .)
+   remove_attributes(data, c("label", "format.spss", "display_width"))
+ }
```

The following function splits a data frame into a list of data frames that contains the variables specified in the `...` plus then each on the other variables:

``` r
> split_by_var <- function(df, ...) {
+   args <- as.list(match.call())
+   var_ref <- unlist(args[-(1:2)])
+   vars <- setdiff(names(df), var_ref)
+   lapply(vars, function(x) df[c(var_ref, x)])
+ }
```

The following function converts a data frame with a categorical variable into a summurized version of it where `table` is called on the categorical variable:

``` r
> summarise_table <- function(df, ...) {
+   require(tidyr)  # spread
+   args <- as.list(match.call())
+   var_ref <- unlist(args[-(1:2)])
+   var <- setdiff(names(df), var_ref)
+   df %>%
+     group_by(.dots = var_ref) %>%
+     count_(var) %>%
+     ungroup() %>%
+     spread(var, n, sep = "_", fill = 0)
+ }
```

The following function combines the two previous functions to reshape a data frame:

``` r
> reshape <- function(df) {
+   require(purrr)  # reduce
+   require(dplyr)  # mutate
+   df %>%
+     split_by_var("District_ID", "Village_ID") %>%
+     lapply(summarise_table, "District_ID", "Village_ID") %>% 
+     reduce(left_join, by = c("District_ID", "Village_ID")) %>% 
+     mutate(code = paste0("1",
+                          sprintf("%02.0f", District_ID),
+                          sprintf("%03.0f", Village_ID)))
+ }
```

The following function replaces values of a vector above a threshold by `NA`s:

``` r
> replace_by_na <- function(x, th) {
+   x[x > th] <- NA
+   x
+ }
```

Reading and summarizing the data by village
-------------------------------------------

### Reading the data

``` r
> households <- read_sav_data("../../raw_data/Lao Statistics Bureau/PHC2015_Household_Record.sav")
> persons <- read_sav_data("../../raw_data/Lao Statistics Bureau/PHC2015_Person_Record_Province1.sav")
```

### Household data

Let's apply the previous two functions to the `households` data set:

``` r
> hh_by_village1 <- households %>%
+   select(District_ID, Village_ID, Household_type,
+          starts_with("Q"),
+          -matches("Q61|2|3"), -matches("Q5[45]")) %>% 
+   reshape()
```

``` r
> hh_by_village2 <- households %>%
+   select(District_ID, Village_ID, matches("Q5[45]")) %>%
+   mutate(Q54._Area_occupied = Q54._Area_occupied %>% 
+            gsub("Not stated", NA, .) %>% 
+            as.integer(),
+          Q55._Number_of_room = Q55._Number_of_room %>% 
+            gsub("Not stated", NA, .) %>% 
+            gsub(" Rooms*", "", .) %>% 
+            as.integer() %>% 
+            replace_by_na(70))
```

``` r
> hh_by_village3 <- hh_by_village2 %>%
+   select(-Q55._Number_of_room) %>% 
+   group_by(District_ID, Village_ID) %>% 
+   summarise(Q54._Area_occupied = mean(Q54._Area_occupied))
```

``` r
> hh_by_village2 %<>%
+   select(-Q54._Area_occupied) %>%
+   reshape()
```

``` r
> hh_by_village4 <- households %>%
+   group_by(District_ID, Village_ID) %>% 
+   tally() %>% 
+   ungroup()
```

``` r
> hh_by_village <- reduce(list(hh_by_village1, hh_by_village2, hh_by_village3, hh_by_village4),
+                         left_join, by = c("District_ID", "Village_ID"))
```

``` r
> write.csv(hh_by_village, "../../cleaned_data/hh_by_village.csv")
```

### Individual data

``` r
> persons1 <- persons %>%
+   select(District_ID, Village_ID, Q5._Age) %>% 
+   bind_cols(select_if(persons, is.factor)) %>% 
+   reshape()
```

``` r
> persons2 <- persons %>%
+   group_by(District_ID, Village_ID) %>% 
+   summarise(
+     nb_person    = n(),
+     nb_household = length(unique(Household_number)),
+     nb_3_4       = sum( 3 <= Q5._Age & Q5._Age < 5),
+     nb_5_8       = sum( 5 <= Q5._Age & Q5._Age < 9),
+     nb_9_12      = sum( 9 <= Q5._Age & Q5._Age < 13),
+     nb_13_18     = sum(13 <= Q5._Age & Q5._Age < 19),
+     nb_19_25     = sum(19 <= Q5._Age & Q5._Age < 26),
+     nb_26_50     = sum(26 <= Q5._Age & Q5._Age < 51),
+     nb_51        = sum(51 <= Q5._Age)
+   )
```

``` r
> prs_by_village <- left_join(persons1, persons2, by = c("District_ID", "Village_ID"))
```

``` r
> write.csv(prs_by_village, "../../cleaned_data/prs_by_village.csv")
```
