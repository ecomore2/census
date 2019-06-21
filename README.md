
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

The 2015 census data come from the [Lao Statistics
Bureau](https://www.lsb.gov.la).

The census is based on this
[questionaire](https://www.dropbox.com/s/6alphptnxl3pjyr/English%20questionnare%20PHC2015.pdf?dl=0)
and its data are in 3 files in the `raw_data/census` folder of the
DropBox Ecomore2 folder:

  - Villages data from `PHC2015 01 province.xlsx` with village
    identifiers and names in Lao and English, and **11 variables** for
    **480 villages**;
  - Households data from `PHC2015_Household_Record.sav` with **168,949
    households** and **33 questions**;
  - Individuals data from `PHC2015_Person_Record_Province1.sav` with
    **820,940 individuals** and **61 questions**.

The cleaning pipeline summarises these 3 data sets per village and merge
them into a data frame of **485 villages** and **260 variables**
available in a CSV file that can be copied and pasted from the
[`data/census.csv`](https://raw.githubusercontent.com/ecomore2/census/master/data/census.csv)
CSV file or can be downloaded directly from R
as:

``` r
> if (! "readr" %in% rownames(installed.packages())) install.packages("readr")
> pacs <- readr::read_csv("https://raw.githubusercontent.com/ecomore2/census/master/data/census.csv",
+                         col_types = paste(c("ddccciiiiilllc", rep("d", 246)), collapse = ""))
```

Below is a description of the variables. The proportions and percentages
variables in the household-based and individual-bases statistics are
computed from categorical variables in the original census data and then
averaged over villages. Quantitative variables in the original census
data are averaged over villages.

  - Unique key:
      - `District_ID`
      - `Village_ID`
  - Village statistics
      - `District_Name`: character string
      - `Village_Name`: character string
      - `village_type`: “Urban”, “Rural with road”, “Rural without road”
      - `private_household`: number
      - `collective_household`: number
      - `male`: number
      - `female`: number
      - `total_pop`: number
      - `water_supply`: boolean
      - `market`: boolean
      - `health_center`: boolean
      - `primaryschool`: “Complete”, “Incomplete”, “No School”
  - Household-based statistics:
      - Household\_type: `collective_households`, `private_households`:
        percentages
      - Q49.\_Tenure\_status\_of\_household: `Employer.provided`,
        `Owner`, `Rent`, `Rentfree`: percentages
      - Q50.\_Roof\_of\_building: `Bamboo_roof`, `Grass`,
        `Tile.sipax.concrete`, `Wood_roof`, `Zinc`: percentages
      - Q51.\_Walls\_of\_building: `Bamboo_wall`, `Brick.concrete`,
        `Wood_wall`: percentages
      - Q52.\_Floor\_of\_building: `Bamboo_floor`, `Ceramic.tile`,
        `Concrete`, `Wood_floor`: percentages
      - Q53.\_Have\_electicity: `No.electricity`, `Own.generator`,
        `Publicly.distributed...own.meter`,
        `Publicly.distributed...shared.meter`, `Using.batteries`:
        percentages
      - Q56.\_Main\_drinking\_water\_source: `Bottled.or.canned.water`,
        `Mountain.source`, `Piped.water`, `Rain.water`,
        `River.stream.dam`, `Tank`, `Well.borehole.protected`,
        `Well.borehole.unprotected`: percentages
      - Q57.\_Distance\_of\_water\_source: `Less.than.200.meters`,
        `On.premises`, `X1000.meters.or.more`, `X200.to.499.meters`,
        `X500.to.999.meters`: percentages
      - Q58.\_Type\_of\_toilet: `Bucket`, `Composting.toilet`,
        `Flush.pour.flush`, `Hang.toilet.hang.latrine`,
        `Pit.latrine.other`, `Pit.latrine.ventilated`: percentages
      - Q59.\_Energy\_for\_cooking: `Charcoal`, `Coal`, `Electricity`,
        `Gas`, `Paraffin.fuel`, `Sawdust`, `Wood_cooking`: percentages
      - `Q36._Death_last_12_month`: proportion
      - `Q40._Moved_in_HH`: proportion
      - `Q44._Moved_out_of_household`: proportion
      - `Q54._Area_occupied`: mean over villages
      - `Q55._Number_of_room`: mean over villages
      - `Q60.1._Tractor`: proportion of households own
      - `Q60.2._Car/van`: proportion of households own
      - `Q60.3._MotorBike`: proportion of households own
      - `Q60.4._Bicycle`: proportion of households own
      - `Q60.5._Boat`: proportion of households own
      - `Q60.6._Radio`: proportion of households own
      - `Q60.7._Television`: proportion of households own
      - `Q60.8._Fixed_phone`: proportion of households own
      - `Q60.9._Cell_phone`: proportion of households own
      - `Q60.10._Computer`: proportion of households own
      - `Q60.11._Washing_machine`: proportion of households own
      - `Q60.12._Air_conditioner`: proportion of households own
      - `Q60.13._Fan`: proportion of households own
      - `Q60.14._Fridge/freezer`: proportion of households own
      - `Q60.15._Agriculture_land`: proportion of households own
      - `Q61._Males`: mean over households
      - `Q62._Females`: mean over households
      - `Q63_Total_persons`: mean over households
      - `area_per_person`: computed from Q54.\_Area\_occupied and
        Q63\_Total\_persons and then averaged over villages
      - `rooms_per_person`: computed from Q55.\_Number\_of\_room and
        Q63\_Total\_persons and then averaged over villages
  - Person-based statistics:
      - Household\_type: `collective_persons`, `private_persons`:
        percentages
      - Q4.\_Sex: `Female`, `Male`: percentages
      - Q5.\_Age: `a0`, …, `a99`: percentages
      - Q6.\_Marital\_Status: `Divorced.separated`, `Married`,
        `Never.married`, `Stay.together`, `Widowed`: percentages
      - Q10A.\_Born\_this\_district: `Other.district.or.abroad`,
        `Same.district`: percentages
      - Q18.\_Can\_read\_and\_write: `Not.at.all`, `Yes..Lao`,
        `Yes..other`: percentages
      - Q19.\_Attended\_school: `Attended.before`, `Attending`, `Never`:
        percentages
      - Q20.\_Highest\_education: `No.grade`, `Grade.1`, `Grade.2`,
        `Grade.3`, `Grade.4`, `Grade.5`, `Grade.6`, `Lower.secondary.1`,
        `Lower.secondary.2`, `Lower.secondary.3`, `Lower.secondary.4`,
        `Upper.secondary.1`, `Upper.secondary.2`, `Upper.secondary.3`,
        `Vocation.education.first.level`,
        `Vocation.education.middle.level`,
        `Vocation.education.high.level`, `Graduate.degree.holder`,
        `Post.graduate.masters.degree`, `Higher.than.post.graduate`:
        percentages
      - Q21.\_Main\_subject\_of\_study: `Agriculture,
        forestry.and.fishing`, `Architecture.and.building`, `Arts`,
        `Business.and.administration`, `Computer`,
        `Engineering.and.trade`, `Environmental.protection`, `Health`,
        `Humanities`, `Journalism.and.information`, `Law`,
        `Life.science`, `Manufacturing.and.processing`,
        `Mathematics.and.statistics`, `Personal.services`,
        `Physical.science`, `Security.services`, `Social.science`,
        `Social.services`, `Teacher.training.and.education.science`,
        `Transportation.services`, `Veterinary`: percentages
      - Q22A.\_Place\_living\_2005: `Other.District.or.abroad`,
        `Same.District`: percentages
      - Q24.\_Main\_activity\_12\_month: `Employer`,
        `Government.employee`, `Household.duties`,
        `International.or.NGO`, `Own.account.worker`,
        `Private.employee`, `State.enterprise.employee`, `Student`,
        `Unemployed`, `Unpaid.family.worker`: percentages
      - Q32M\_Age\_first\_birth: number

Note that some variables are thus available in different versions
depending on whether they have been averaged from individual or
household data. This is the case for the 4 variables listed below:

| household-based         | person-based         |
| ----------------------- | -------------------- |
| `Q61._Males`            | `Male`               |
| `Q62._Females`          | `Female`             |
| `collective_households` | `collective_persons` |
| `private_households`    | `private_persons`    |
