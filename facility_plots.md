---
title: "ICE Detention Data FY21"
author: Nathan Craig
date: 15 Apr 2021
output:
  html_document:
    df_print: paged
    toc: true
    keep_md: yes
  html_notebook:
    toc: true
---
# Introduction
This document represents exploratory data analysis of ICE's Fiscal Year End Detention Reports. Currently, I am focusing on the FY21 dataset.


```r
library(sf)
```

```
## Linking to GEOS 3.9.0, GDAL 3.2.1, PROJ 7.2.1
```

```r
library(ggplot2)
library(readxl)
library(tidyverse)
```

```
## -- Attaching packages --------------------------------------- tidyverse 1.3.0 --
```

```
## v tibble  3.1.0     v dplyr   1.0.5
## v tidyr   1.1.3     v stringr 1.4.0
## v readr   1.4.0     v forcats 0.5.1
## v purrr   0.3.4
```

```
## -- Conflicts ------------------------------------------ tidyverse_conflicts() --
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

```r
library(summarytools)
```

```
## Registered S3 method overwritten by 'pryr':
##   method      from
##   print.bytes Rcpp
```

```
## 
## Attaching package: 'summarytools'
```

```
## The following object is masked from 'package:tibble':
## 
##     view
```

```r
library(knitr)
opts_chunk$set(results = 'asis',
               comment = NA,
               prompt = FALSE,
               cache = FALSE)
```



```r
st_options(plain.ascii = FALSE,
           footnote = NA,
           subtitle.emphasis = FALSE)
```



## Load Data
Data were downloaded from the [ICE Detention Management](https://www.ice.gov/detain/detention-management) webpage.


Based on facility address, coordinates were established using [Geocodio](https://www.geocod.io/), appended to the spreadsheet and converted to a shapefile. This work was done using ArcGIS.


```r
facilities <- st_read("./map_data/Facilities_FY21.shp")
```

Reading layer `Facilities_FY21' from data source `C:\Users\nmc\Nextcloud\AVID_Restricted\ICE_data\IDE-Detention-Year-End-Reports\map_data\Facilities_FY21.shp' using driver `ESRI Shapefile'
Simple feature collection with 142 features and 39 fields
Geometry type: POINT
Dimension:     XY
Bounding box:  xmin: -157.9283 ymin: 13.44426 xmax: 145.7408 ymax: 61.30192
Geodetic CRS:  NAD83

```r
states <- st_read("./map_data/cb_2018_us_state_500k.shp")
```

Reading layer `cb_2018_us_state_500k' from data source `C:\Users\nmc\Nextcloud\AVID_Restricted\ICE_data\IDE-Detention-Year-End-Reports\map_data\cb_2018_us_state_500k.shp' using driver `ESRI Shapefile'
Simple feature collection with 56 features and 9 fields
Geometry type: MULTIPOLYGON
Dimension:     XY
Bounding box:  xmin: -179.1489 ymin: -14.5487 xmax: 179.7785 ymax: 71.36516
Geodetic CRS:  NAD83


```r
facilities <- rename(facilities,
       `Accuracy Score` = Accuracy_S,
       `Accuracy Type` = Accuracy_T,
       `Zip Geocoded` = Zip_Geocod,
       `Type Detailed` = Type_Detai,
       `Male/Female` = Male_Femal,
       `Level A` = Level_A,
       `Level B` = Level_B,
       `Level C` = Level_C,
       `Level D` = Level_D,
       `Male Crim` = Male_Crim,
       `Male Non Crim` = Male_Non_C,
       `Female Crim` = Female_Cri,
       `Female Non Crim` = Female_Non,
       `ICE Threat Level 1` = ICE_Threat,
       `ICE Threat Level 2` = ICE_Thre_1,
       `ICE Threat Level 3` = ICE_Thre_2,
       `ICE No Threat Level` = No_ICE_Thr,
       `Mandatory Detention` = Mandatory,
       `Guaranteed Minimum (character)` = Guaranteed,
       `Guaranteed Minimum` = Guarante_1,
       `Last Inspection Type` = Last_Inspe,
       `Last Inspection Standard` = Last_Ins_1,
       `Last Inspection Rating - Final` = Last_Ins_2,
       `Last Inspection Date` = Last_Ins_3,
       `Second to Last Inspection Type` = Second_to,
       `Second to Last Inspection Standard` = Second_t_1,
       `Second to Last Inspection Rating` = Second_t_2,
       `Second to Last Insepction Date` = Second_t_3,
  ) %>% 
  relocate(`Guaranteed Minimum`, .after = `Guaranteed Minimum (character)`) %>% 
  select(-`Guaranteed Minimum (character)`)
```








# Summary Statistics

Calculate detention totals.


```r
facilities <- facilities %>% 
  mutate(`Total Detained`=
           `Female Crim` +
           `Female Non Crim` +
           `Male Crim` +
           `Male Non Crim`) %>% 
  mutate(`Total Males Detained` =
           `Male Crim` +
           `Male Non Crim`) %>% 
  mutate(`Total Females Detained` =
           `Female Crim` +
           `Female Non Crim`) %>% 
  mutate(`Percent No Threat` =
            (`ICE No Threat Level` /
           `Total Detained`)*100) %>% 
  relocate(c(`Total Females Detained`,
             `Total Males Detained`,
             `Total Detained`,
             `Percent No Threat`), .after = `Guaranteed Minimum`)
```



```r
facilities %>% 
  select(c(-`Accuracy Score`, -Longitude, -Latitude, -Zip, -`Zip Geocoded`)) %>% 
  descr(transpose = TRUE, stats = c("mean", "sd", "min", "med", "max"), headings = FALSE)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["Mean"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["Std.Dev"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["Min"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["Median"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["Max"],"name":[5],"type":["dbl"],"align":["right"]}],"data":[{"1":"3.063380","2":"6.773137","3":"0","4":"0.00000","5":"51","_rn_":"Female Crim"},{"1":"9.161972","2":"28.160038","3":"0","4":"0.00000","5":"188","_rn_":"Female Non Crim"},{"1":"65.126761","2":"67.906889","3":"0","4":"53.00000","5":"379","_rn_":"FY21_ALOS"},{"1":"687.800000","2":"493.902246","3":"2","4":"580.00000","5":"2400","_rn_":"Guaranteed Minimum"},{"1":"54.140845","2":"87.183884","3":"0","4":"14.00000","5":"513","_rn_":"ICE No Threat Level"},{"1":"31.683099","2":"45.518038","3":"0","4":"14.00000","5":"276","_rn_":"ICE Threat Level 1"},{"1":"12.035211","2":"16.017018","3":"0","4":"6.00000","5":"98","_rn_":"ICE Threat Level 2"},{"1":"11.464789","2":"15.734347","3":"0","4":"6.00000","5":"94","_rn_":"ICE Threat Level 3"},{"1":"45.767606","2":"84.258944","3":"0","4":"9.00000","5":"507","_rn_":"Level A"},{"1":"16.746479","2":"26.200383","3":"0","4":"6.50000","5":"156","_rn_":"Level B"},{"1":"23.795775","2":"33.450405","3":"0","4":"10.00000","5":"183","_rn_":"Level C"},{"1":"22.957746","2":"34.318672","3":"0","4":"10.00000","5":"181","_rn_":"Level D"},{"1":"52.281690","2":"71.067181","3":"0","4":"26.50000","5":"466","_rn_":"Male Crim"},{"1":"44.894366","2":"76.743536","3":"0","4":"12.00000","5":"513","_rn_":"Male Non Crim"},{"1":"74.732394","2":"95.626113","3":"0","4":"35.00000","5":"526","_rn_":"Mandatory Detention"},{"1":"37.796837","2":"27.708575","3":"0","4":"31.01604","5":"100","_rn_":"Percent No Threat"},{"1":"109.401408","2":"138.545436","3":"0","4":"54.50000","5":"734","_rn_":"Total Detained"},{"1":"12.225352","2":"31.367733","3":"0","4":"0.00000","5":"195","_rn_":"Total Females Detained"},{"1":"97.176056","2":"129.322857","3":"0","4":"48.00000","5":"734","_rn_":"Total Males Detained"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>


## Guaranteed Minimums
Of the 142 facilities listed in the FY21 ICE Detention Year End Report 35% (n = 50) have Guaranteed Minimums as part of their contracts. The table below lists those facilities in descending order based on the Guaranteed Minimum number of beds stipulated in the contract.


```r
facilities %>% 
  as_tibble() %>% 
  select(c(Name, `Guaranteed Minimum`, `Total Detained`, `Mandatory Detention`, `Percent No Threat`)) %>% 
  filter(`Guaranteed Minimum` >0) %>% 
  arrange(desc(`Guaranteed Minimum`))
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Name"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Guaranteed Minimum"],"name":[2],"type":["int"],"align":["right"]},{"label":["Total Detained"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["Mandatory Detention"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["Percent No Threat"],"name":[5],"type":["dbl"],"align":["right"]}],"data":[{"1":"South Texas Family Residential Center","2":"2400","3":"299","4":"155","5":"99.331104"},{"1":"La Palma Correctional Center","2":"1800","3":"428","4":"248","5":"77.570093"},{"1":"La Palma Correction Center - Apso","2":"1800","3":"378","4":"247","5":"69.312169"},{"1":"Stewart Detention Center","2":"1600","3":"673","4":"526","5":"30.609212"},{"1":"Adelanto ICE Processing Center","2":"1455","3":"336","4":"221","5":"12.797619"},{"1":"South Texas ICE Processing Center","2":"1350","3":"734","4":"464","5":"69.891008"},{"1":"Tacoma ICE Processing Center (Northwest Det Ctr)","2":"1181","3":"311","4":"248","5":"16.398714"},{"1":"Lasalle ICE Processing Center (Jena)","2":"1170","3":"464","4":"344","5":"55.172414"},{"1":"Otay Mesa Detention Center (San Diego Cdf)","2":"1100","3":"352","4":"227","5":"53.977273"},{"1":"Adams County Det Center","2":"1100","3":"348","4":"318","5":"65.804598"},{"1":"Winn Correctional Center","2":"946","3":"365","4":"301","5":"57.260274"},{"1":"Karnes County Residential Center","2":"830","3":"112","4":"34","5":"100.000000"},{"1":"Port Isabel","2":"800","3":"392","4":"190","5":"83.163265"},{"1":"Jackson Parish Correctional Center","2":"751","3":"184","4":"144","5":"72.826087"},{"1":"Bluebonnet Detention Facility","2":"750","3":"374","4":"227","5":"31.016043"},{"1":"Montgomery ICE Processing Center","2":"750","3":"329","4":"207","5":"46.504559"},{"1":"El Valle Detention Facility","2":"750","3":"327","4":"175","5":"83.486239"},{"1":"Houston Contract Detention Facility","2":"750","3":"190","4":"124","5":"58.947368"},{"1":"Torrance County Detention Facility","2":"714","3":"24","4":"21","5":"41.666667"},{"1":"Broward Transitional Center","2":"700","3":"331","4":"193","5":"77.643505"},{"1":"South Louisiana Detention Center","2":"700","3":"84","4":"73","5":"71.428571"},{"1":"Richwood Correctional Center","2":"677","3":"156","4":"134","5":"80.128205"},{"1":"Imperial Regional Detention Facility","2":"640","3":"314","4":"203","5":"79.936306"},{"1":"El Paso Service Processing Center","2":"600","3":"323","4":"159","5":"67.801858"},{"1":"Irwin County Detention Center","2":"600","3":"314","4":"248","5":"31.847134"},{"1":"Golden State Annex","2":"560","3":"83","4":"64","5":"3.614458"},{"1":"Prairieland Detention Facility","2":"550","3":"292","4":"214","5":"31.849315"},{"1":"Folkston Main Ipc","2":"544","3":"123","4":"50","5":"39.837398"},{"1":"Denver Contract Detention Facility","2":"525","3":"218","4":"154","5":"19.724771"},{"1":"York County Prison","2":"500","3":"327","4":"250","5":"22.935780"},{"1":"Otero County Processing Center","2":"500","3":"188","4":"125","5":"55.851064"},{"1":"Immigration Centers Of America Farmville","2":"500","3":"101","4":"70","5":"15.841584"},{"1":"T. Don Hutto Detention Center","2":"461","3":"71","4":"52","5":"98.591549"},{"1":"Krome North Service Processing Center","2":"450","3":"310","4":"217","5":"30.967742"},{"1":"Denver Contract Detention Facility (Cdf) Ii","2":"432","3":"38","4":"28","5":"10.526316"},{"1":"Buffalo (Batavia) Service Processing Center","2":"400","3":"252","4":"218","5":"18.650794"},{"1":"Florence Service Processing Center","2":"392","3":"72","4":"44","5":"61.111111"},{"1":"River Correctional Center","2":"361","3":"140","4":"109","5":"80.000000"},{"1":"Iah Secure Adult Detention Facility (Polk)","2":"350","3":"76","4":"64","5":"60.526316"},{"1":"Mesa Verde ICE Processing Center","2":"320","3":"43","4":"39","5":"2.325581"},{"1":"Glades County Detention Center","2":"300","3":"257","4":"171","5":"28.015564"},{"1":"Elizabeth Contract Detention Facility","2":"285","3":"102","4":"58","5":"75.490196"},{"1":"Rio Grande Detention Center","2":"275","3":"155","4":"118","5":"69.677419"},{"1":"Caroline Detention Facility","2":"224","3":"176","4":"108","5":"19.318182"},{"1":"Yuba County Jail","2":"150","3":"19","4":"17","5":"0.000000"},{"1":"Desert View","2":"120","3":"13","4":"11","5":"15.384615"},{"1":"Allen Parish Public Safety Complex","2":"100","3":"64","4":"55","5":"84.375000"},{"1":"San Luis Regional Detention Center","2":"100","3":"54","4":"31","5":"64.814815"},{"1":"Calhoun County Correctional Center","2":"75","3":"114","4":"83","5":"18.421053"},{"1":"Northwestern Regional Juvenile Detention Center","2":"2","3":"0","4":"0","5":"NaN"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

## Estimated number of "Ghost Beds"

```r
facilities %>% 
  as_tibble() %>% 
  select(c(Name, `Guaranteed Minimum`, `Total Detained`, `Percent No Threat`)) %>% 
  filter(`Guaranteed Minimum` >0) %>% 
  mutate(`Estimated Ghost Beds` = `Guaranteed Minimum` - `Total Detained`) %>% 
  select(c(Name, `Estimated Ghost Beds`)) %>% 
  arrange(desc(`Estimated Ghost Beds`))
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Name"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Estimated Ghost Beds"],"name":[2],"type":["dbl"],"align":["right"]}],"data":[{"1":"South Texas Family Residential Center","2":"2101"},{"1":"La Palma Correction Center - Apso","2":"1422"},{"1":"La Palma Correctional Center","2":"1372"},{"1":"Adelanto ICE Processing Center","2":"1119"},{"1":"Stewart Detention Center","2":"927"},{"1":"Tacoma ICE Processing Center (Northwest Det Ctr)","2":"870"},{"1":"Adams County Det Center","2":"752"},{"1":"Otay Mesa Detention Center (San Diego Cdf)","2":"748"},{"1":"Karnes County Residential Center","2":"718"},{"1":"Lasalle ICE Processing Center (Jena)","2":"706"},{"1":"Torrance County Detention Facility","2":"690"},{"1":"South Texas ICE Processing Center","2":"616"},{"1":"South Louisiana Detention Center","2":"616"},{"1":"Winn Correctional Center","2":"581"},{"1":"Jackson Parish Correctional Center","2":"567"},{"1":"Houston Contract Detention Facility","2":"560"},{"1":"Richwood Correctional Center","2":"521"},{"1":"Golden State Annex","2":"477"},{"1":"El Valle Detention Facility","2":"423"},{"1":"Montgomery ICE Processing Center","2":"421"},{"1":"Folkston Main Ipc","2":"421"},{"1":"Port Isabel","2":"408"},{"1":"Immigration Centers Of America Farmville","2":"399"},{"1":"Denver Contract Detention Facility (Cdf) Ii","2":"394"},{"1":"T. Don Hutto Detention Center","2":"390"},{"1":"Bluebonnet Detention Facility","2":"376"},{"1":"Broward Transitional Center","2":"369"},{"1":"Imperial Regional Detention Facility","2":"326"},{"1":"Florence Service Processing Center","2":"320"},{"1":"Otero County Processing Center","2":"312"},{"1":"Denver Contract Detention Facility","2":"307"},{"1":"Irwin County Detention Center","2":"286"},{"1":"El Paso Service Processing Center","2":"277"},{"1":"Mesa Verde ICE Processing Center","2":"277"},{"1":"Iah Secure Adult Detention Facility (Polk)","2":"274"},{"1":"Prairieland Detention Facility","2":"258"},{"1":"River Correctional Center","2":"221"},{"1":"Elizabeth Contract Detention Facility","2":"183"},{"1":"York County Prison","2":"173"},{"1":"Buffalo (Batavia) Service Processing Center","2":"148"},{"1":"Krome North Service Processing Center","2":"140"},{"1":"Yuba County Jail","2":"131"},{"1":"Rio Grande Detention Center","2":"120"},{"1":"Desert View","2":"107"},{"1":"Caroline Detention Facility","2":"48"},{"1":"San Luis Regional Detention Center","2":"46"},{"1":"Glades County Detention Center","2":"43"},{"1":"Allen Parish Public Safety Complex","2":"36"},{"1":"Northwestern Regional Juvenile Detention Center","2":"2"},{"1":"Calhoun County Correctional Center","2":"-39"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>


```r
# Not currently using, but a working example is provided.

# facilities_crop <- st_crop(facilities, xmin = -125, xmax = -64,
#                                       ymin = 16.5, ymax = 49.5)
# 
# states_crop <-  st_crop(states, xmin = -125, xmax = -64,
#                                       ymin = 16.5, ymax = 49.5)
```



```r
main_map <- ggplot()+
  geom_sf(data=states)+
  geom_sf(data=facilities, aes(color = `Guaranteed Minimum`, size = `Mandatory Detention`))+
  # ggtitle(,
  #         )+
  coord_sf()

# Zoom the map. Note this only seems to work when called separately.
main_map +
  coord_sf(xlim = c(-125,-64), ylim = c(16.5, 49.5), expand = FALSE)+
  labs(
    title = "ICE Detention Facilities",
    subtitle = "Showing Mandatory Detention and Contractual Guaranteed Minimums",
    caption = "Data source: https://www.ice.gov/detain/detention-management",
    color ="Guaranteed\nMinimum",
    size = "Mandatory\nDetention")
```

```
Coordinate system already present. Adding new coordinate system, which will replace the existing one.
```

![](facility_plots_files/figure-html/facility map with mandatory detention and guaranteed minimums-1.png)<!-- -->

# Not Used





```r
df <- df %>% 
  mutate(`Total Detained`=
           `Female Crim` +
           `Female Non-Crim` +
           `Male Crim` +
           `Male Non-Crim`) %>% 
  mutate(`Total Males Detained` =
           `Male Crim` +
           `Male Non-Crim`) %>% 
  mutate(`Total Females Detained` =
           `Female Crim` +
           `Female Non-Crim`) %>% 
  mutate(`Percent No Threat` =
            (`No ICE Threat Level` /
           `Total Detained`)*100)
```


```r
descr(df, transpose = TRUE, stats = c("mean", "sd", "min", "med", "max"), headings = FALSE)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["Mean"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["Std.Dev"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["Min"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["Median"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["Max"],"name":[5],"type":["dbl"],"align":["right"]}],"data":[{"1":"3.063380","2":"6.773137","3":"0","4":"0.00000","5":"51","_rn_":"Female Crim"},{"1":"9.161972","2":"28.160038","3":"0","4":"0.00000","5":"188","_rn_":"Female Non-Crim"},{"1":"66.057143","2":"67.939644","3":"1","4":"53.00000","5":"379","_rn_":"FY21 ALOS"},{"1":"687.800000","2":"493.902246","3":"2","4":"580.00000","5":"2400","_rn_":"Guaranteed Minimum"},{"1":"31.683099","2":"45.518038","3":"0","4":"14.00000","5":"276","_rn_":"ICE Threat Level 1"},{"1":"12.035211","2":"16.017018","3":"0","4":"6.00000","5":"98","_rn_":"ICE Threat Level 2"},{"1":"11.464789","2":"15.734347","3":"0","4":"6.00000","5":"94","_rn_":"ICE Threat Level 3"},{"1":"45.767606","2":"84.258944","3":"0","4":"9.00000","5":"507","_rn_":"Level A"},{"1":"16.746479","2":"26.200383","3":"0","4":"6.50000","5":"156","_rn_":"Level B"},{"1":"23.795775","2":"33.450405","3":"0","4":"10.00000","5":"183","_rn_":"Level C"},{"1":"22.957746","2":"34.318672","3":"0","4":"10.00000","5":"181","_rn_":"Level D"},{"1":"52.281690","2":"71.067181","3":"0","4":"26.50000","5":"466","_rn_":"Male Crim"},{"1":"44.894366","2":"76.743536","3":"0","4":"12.00000","5":"513","_rn_":"Male Non-Crim"},{"1":"74.732394","2":"95.626113","3":"0","4":"35.00000","5":"526","_rn_":"Mandatory"},{"1":"54.140845","2":"87.183884","3":"0","4":"14.00000","5":"513","_rn_":"No ICE Threat Level"},{"1":"37.796837","2":"27.708575","3":"0","4":"31.01604","5":"100","_rn_":"Percent No Threat"},{"1":"43514.230159","2":"703.608836","3":"39241","4":"43695.50000","5":"44265","_rn_":"Second to Last Inspection Date"},{"1":"109.401408","2":"138.545436","3":"0","4":"54.50000","5":"734","_rn_":"Total Detained"},{"1":"12.225352","2":"31.367733","3":"0","4":"0.00000","5":"195","_rn_":"Total Females Detained"},{"1":"97.176056","2":"129.322857","3":"0","4":"48.00000","5":"734","_rn_":"Total Males Detained"},{"1":"57841.605634","2":"28335.003901","3":"939","4":"69732.00000","5":"99577","_rn_":"Zip"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>
