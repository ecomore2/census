
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

The census data are 2 .sav files in the `raw_data/census` folder of the
DropBox Ecomore2 folder: `PHC2015_Household_Record.sav` and
`PHC2015_Person_Record_Province1.sav`. The first one contains data by
household with **168,949 households** and **33 questions** whereas the
second one contains data per person with **820,940 individuals** and
**61 questions**. Here we summarise both data set per village and the
output is a table of **482 villages** and **1,244** columns that is
written in the `census.csv` and `census.rds` files of the `cleaned_data`
folder of the DropBox Ecomore2 folder. For a quick overview of the
information available in the data, see below the types of questions
asked, ordered into 6 main categories (general individual questions,
handicaps, education, movements, motherhood, type of household).

**1 - General individual questions:**

| Q   | question                                  |
| --- | ----------------------------------------- |
| \-  | type of household (private vs collective) |
| \-  | type of village (urban vs rural)          |
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
| --- | ---------------------- |
| Q11 | seeing handicap        |
| Q12 | hearing handicap       |
| Q13 | walking handicap       |
| Q14 | focusing handicap      |
| Q15 | hygiene handicap       |
| Q16 | communication handicap |
| Q17 | cause of handicap      |

**3 - Education:**

| Q   | question          |
| --- | ----------------- |
| Q18 | literacy          |
| Q19 | school attendance |
| Q20 | education level   |
| Q21 | subject of study  |

**4 - Movements:**

| Q   | question                                              |
| --- | ----------------------------------------------------- |
| Q10 | district of birth                                     |
| Q22 | district living in 2005                               |
| Q23 | reason for moving                                     |
| Q40 | moved in household (concerns the whole household)     |
| Q42 | district living in 2014                               |
| Q44 | moved out of household (concerns the whole household) |

**5 - Motherhood:**

| Q   | question                  |
| --- | ------------------------- |
| Q28 | birth history             |
| Q29 | children with you         |
| Q30 | children elsewhere        |
| q31 | children dead             |
| Q32 | age at first birth        |
| Q33 | year and month last birth |
| Q34 | sex last birth            |
| Q35 | birth alive               |

**6 - Type of household:**

| Q   | question                                  |
| --- | ----------------------------------------- |
| \-  | type of household (private vs collective) |
| Q36 | death last 12 months                      |
| Q49 | tenure status of household                |
| Q50 | type of roof                              |
| Q51 | type of walls                             |
| Q52 | type of floor                             |
| Q53 | electricity status                        |
| Q54 | mean area per person                      |
| Q55 | number of rooms                           |
| Q56 | source of drinking water                  |
| Q57 | distance to water source                  |
| Q58 | type of toilet                            |
| Q59 | type of cooking energy                    |
| Q60 | possessions (phone, motorbike, TV, etc…)  |
| Q63 | number of people per household            |

We additionally computed the following 3 variables:

  - number of individuals per age class
  - area per individual within household
  - number of room per individual within household

Possession concerns tractor, car/van, motorbike, bicycle, boat, radio,
TV, landline phone, cell phone, computer, washing machine, AC, fan,
fridge/freezer and agricultural land.
