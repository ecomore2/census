
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

The census data are 2 .sav files in the `raw_data/census` folder of the DropBox Ecomore2 folder: `PHC2015_Household_Record.sav` and `PHC2015_Person_Record_Province1.sav`. The first one contains data by household with **168,949 households** and **33 questions** whereas the second one contains data per person with **820,940 individuals** and **61 questions**. Here we summarise both data set per village and the output is a table of **482 villages** and **1,244** columns that is written in the `census.csv` and `census.rds` files of the `cleaned_data` folder of the DropBox Ecomore2 folder. For a quick overview of the information available in the data, see below the types of questions asked, ordered into 6 main categories (general individual questions, handicaps, education, movements, motherhood, type of household).

**1 - General individual questions:**

| Q   | question                                  |
|-----|-------------------------------------------|
| -   | type of household (private vs collective) |
| -   | type of village (urban vs rural)          |
| Q3  | relationship to the head of the household |
| Q4  | sex                                       |
| Q5  | age                                       |
| Q6  | marital status                            |
| Q7  | citizenship                               |
| Q8  | ethnicity                                 |
| Q9  | religion                                  |
| Q24 | main activity during last 12 months       |

**2 - Handicaps:**

| Q   | question               |
|-----|------------------------|
| Q11 | seeing handicap        |
| Q12 | hearing handicap       |
| Q13 | walking handicap       |
| Q14 | focusing handicap      |
| Q15 | hygiene handicap       |
| Q16 | communication handicap |
| Q17 | cause of handicap      |

**3 - Education:**

| Q   | question          |
|-----|-------------------|
| Q18 | literacy          |
| Q19 | school attendance |
| Q20 | education level   |
| Q21 | subject of study  |

**4 - Movements:**

| Q   | question                                              |
|-----|-------------------------------------------------------|
| Q10 | district of birth                                     |
| Q22 | district living in 2005                               |
| Q23 | reason for moving                                     |
| Q40 | moved in household (concerns the whole household)     |
| Q42 | district living in 2014                               |
| Q44 | moved out of household (concerns the whole household) |

**5 - Motherhood:**

| Q   | question                  |
|-----|---------------------------|
| Q28 | birth history             |
| Q29 | children with you         |
| Q30 | children elsewhere        |
| q31 | children dead             |
| Q32 | age at first birth        |
| Q33 | year and month last birth |
| Q34 | sex last birth            |
| Q35 | birth alive               |

**6 - Type of household:**

| Q   | question                                   |
|-----|--------------------------------------------|
| -   | type of household (private vs collective)  |
| Q36 | death last 12 months                       |
| Q49 | tenure status of household                 |
| Q50 | type of roof                               |
| Q51 | type of walls                              |
| Q52 | type of floor                              |
| Q53 | electricity status                         |
| Q54 | mean area per person                       |
| Q55 | number of rooms                            |
| Q56 | source of drinking water                   |
| Q57 | distance to water source                   |
| Q58 | type of toilet                             |
| Q59 | type of cooking energy                     |
| Q60 | possessions (phone, motorbike, TV, etc...) |
| Q63 | number of people per household             |

We additionally computed the following 3 variables:

-   number of individuals per age class
-   area per individual within household
-   number of room per individual within household

Possession concerns tractor, car/van, motorbike, bicycle, boat, radio, TV, landline phone, cell phone, computer, washing machine, AC, fan, fridge/freezer and agricultural land.

Packages
--------

Packages currently installed on the system:

``` r
> installed_packages <- rownames(installed.packages())
```

Packages that we need from [CRAN](https://cran.r-project.org):

``` r
> cran <- c("devtools", # development tools
+           "dplyr",    # manipulating data frames
+           "haven",    # importing .sav files
+           "labelled", # manipulating labelled data
+           "purrr"     # functional programming tools
+           )
```

Installing these packages when not already installed:

``` r
> to_install <- !cran %in% installed_packages
> if (any(to_install)) install.packages(cran[to_install])
```

We additionally need the `ecomore` package from [GitHub](https://github.com/ecomore2/ecomore):

``` r
> if (! "ecomore" %in% installed_packages)  devtools::install_github("ecomore2/ecomore")
```

Loading the packages for interactive use at the command line:

``` r
> invisible(lapply(c(setdiff(cran, "devtools"), "ecomore"), library, character.only = TRUE))
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
+     spread(var, n, sep = "_", fill = 0) %>% 
+     ungroup()
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
+     reduce(left_join, by = c("District_ID", "Village_ID"))
+ }
```

The following function replaces values of a vector above a threshold by `NA`s:

``` r
> replace_by_na <- function(x, th) {
+   x[x > th] <- NA
+   x
+ }
```

The following function calculates the proportion of people coming from out of their current district. It works on the `Q22A._Place_living_2005` or the `Q42AM_Moved_from_same_District` variables of the individual data set.

``` r
> proportion_different_district <- function(x) {
+   x %>% 
+     table() %>%
+     as.matrix() %>%
+     t() %>%
+     as.data.frame() %>% 
+     setNames(tolower(names(.))) %$% 
+     {`other district or abroad` / (`other district or abroad` + `same district`)}
+ }
```

The following function converts the `NaN` values of a vector to `NA`s:

``` r
> nan2na <- function(x) {
+   x[is.nan(x)] <- NA
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

Removing the number of males (`Q61._Males`, `Q61X_Males_4_positions`), females (`Q62._Females`, `Q62X_Females_4_positions`) and people (`Q63_Total_persons`, `Q63X_Total_persons_4_positions`), plus additional cleaning:

``` r
> households %<>% select(District_ID, Village_ID, Household_type,
+                          starts_with("Q"), -matches("Q6[1|2]"),
+                          -Q63X_Total_persons_4_positions) %>%
+                 mutate(Q54._Area_occupied = Q54._Area_occupied %>% 
+                          gsub("Not stated", NA, .) %>% 
+                          as.integer(),
+                        Q55._Number_of_room = Q55._Number_of_room %>% 
+                          gsub("Not stated", NA, .) %>% 
+                          gsub(" Rooms*", "", .) %>% 
+                          as.integer() %>% 
+                          replace_by_na(70),
+                        Q63_Total_persons = Q63_Total_persons %>% 
+                          as.character() %>% 
+                          as.integer() %>% 
+                          replace_by_na(20),
+                        area_per_pers = Q54._Area_occupied / Q63_Total_persons,
+                        nb_rooms_per_pers = Q55._Number_of_room / Q63_Total_persons)
```

Reshaping the categorical variables:

``` r
> households1 <- households %>% 
+   select(-Q54._Area_occupied, -area_per_pers, -nb_rooms_per_pers) %>% 
+   reshape()
```

Computing the mean occupied area and the number of households per village:

``` r
> households2 <- households %>%
+   group_by(District_ID, Village_ID) %>% 
+   summarise(Q54._Area_occupied     = mean(Q54._Area_occupied, na.rm = TRUE),
+             mean_area_per_pers     = mean(area_per_pers, na.rm = TRUE),
+             mean_nb_rooms_per_pers = mean(nb_rooms_per_pers,  na.rm = TRUE),
+             nb_households          = n()) %>% 
+   ungroup()
```

### Individual data

Reshaping the categorical variables:

``` r
> persons1 <- persons %>%
+   select(District_ID, Village_ID, Q5._Age) %>%
+ # adding all the factors except Q26 and Q27:
+   bind_cols(., select_if(persons, is.factor) %>%
+                  select(-Q26._Industry_of_working, -Q27M_Person_Id_Fertility)) %>%
+   mutate(age = cut(Q5._Age, c(3, 5, 9, 13, 19, 26, 51, max(Q5._Age) + 1), right = FALSE)) %>%
+   reshape()
```

Generating the total number of people:

``` r
> persons2 <- persons %>%
+   group_by(District_ID, Village_ID) %>% 
+   summarise(nb_person = n())
```

### Writing the CSV and RDS files

Putting individual and household aggregated data together and writting to disk:

``` r
> list(households1, households2, persons1, persons2) %>% 
+   reduce(left_join, by = c("District_ID", "Village_ID")) %>%
+   mutate(code = paste0("1", sprintf("%02.0f", District_ID),
+                             sprintf("%03.0f", Village_ID))) %>% 
+   filter(Village_ID != 999) %>% 
+   mutate_at(vars(-matches("per_pers|code")), as.integer) %>% 
+   assign("census", ., envir = .GlobalEnv) %>% 
+   write2disk("cleaned_data", "census")
```

### Tests

``` r
> anti_join(persons1,    persons2,    c("District_ID", "Village_ID")) %>% nrow()
[1] 0
> anti_join(persons1,    households1, c("District_ID", "Village_ID")) %>% nrow()
[1] 0
> anti_join(persons1,    households2, c("District_ID", "Village_ID")) %>% nrow()
[1] 0
> anti_join(persons2,    persons1,    c("District_ID", "Village_ID")) %>% nrow()
[1] 0
> anti_join(persons2,    households1, c("District_ID", "Village_ID")) %>% nrow()
[1] 0
> anti_join(persons2,    households2, c("District_ID", "Village_ID")) %>% nrow()
[1] 0
> anti_join(households1, households2, c("District_ID", "Village_ID")) %>% nrow()
[1] 0
> anti_join(households1, persons1,    c("District_ID", "Village_ID")) %>% nrow()
[1] 0
> anti_join(households1, persons2,    c("District_ID", "Village_ID")) %>% nrow()
[1] 0
> anti_join(households2, households2, c("District_ID", "Village_ID")) %>% nrow()
[1] 0
> anti_join(households2, persons1,    c("District_ID", "Village_ID")) %>% nrow()
[1] 0
> anti_join(households2, persons2,    c("District_ID", "Village_ID")) %>% nrow()
[1] 0
```
