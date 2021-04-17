---
title: "ICE Detention Data FY21"
author: Nathan Craig
date: 15 Apr 2021
output:
  html_document:
    df_print: paged
    toc: true
    number_sections: true
    keep_md: yes
  html_notebook:
    toc: true
bibliography: references.bib
---

# Introduction

This document represents exploratory data analysis of ICE's Fiscal Year End Detention Reports. Currently, I am focusing on the FY21 dataset.


```r
library(sf)
library(ggplot2)
library(readxl)
library(tidyverse)
library(summarytools)
st_options(plain.ascii = FALSE,
           footnote = NA,
           subtitle.emphasis = FALSE)
library(knitr)
opts_chunk$set(results = 'asis',
               comment = NA,
               prompt = FALSE,
               cache = FALSE)
```

## Load Data

Data were downloaded from the [ICE Detention Management](https://www.ice.gov/detain/detention-management) webpage.

Based on facility address, coordinates were established using [Geocodio](https://www.geocod.io/), appended to the spreadsheet and converted to a shapefile. This work was done using ArcGIS.


```r
# Load Datasets
facilities <- st_read("./map_data/Facilities_FY21.shp")
states <- st_read("./map_data/cb_2018_us_state_500k.shp")
```


```r
# Rename Fields
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

# Convert Field Type
facilities$`Guaranteed Minimum` <- as.numeric(facilities$`Guaranteed Minimum`)
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




Of the 142 facilities listed in the FY21 ICE Detention Year End Report 35% (n = 50) have Guaranteed Minimums as part of their contracts. Based on data reported by ICE, there are a total of 34390 guaranteed beds in FY21. The table below lists those facilities in descending order based on the Guaranteed Minimum number of beds stipulated in the contract.


```r
facilities %>% 
  as_tibble() %>% 
  select(c(Name, `Guaranteed Minimum`, `Total Detained`, `Mandatory Detention`, `Percent No Threat`)) %>% 
  filter(`Guaranteed Minimum` >0) %>% 
  arrange(desc(`Guaranteed Minimum`))
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Name"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Guaranteed Minimum"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["Total Detained"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["Mandatory Detention"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["Percent No Threat"],"name":[5],"type":["dbl"],"align":["right"]}],"data":[{"1":"South Texas Family Residential Center","2":"2400","3":"299","4":"155","5":"99.331104"},{"1":"La Palma Correctional Center","2":"1800","3":"428","4":"248","5":"77.570093"},{"1":"La Palma Correction Center - Apso","2":"1800","3":"378","4":"247","5":"69.312169"},{"1":"Stewart Detention Center","2":"1600","3":"673","4":"526","5":"30.609212"},{"1":"Adelanto ICE Processing Center","2":"1455","3":"336","4":"221","5":"12.797619"},{"1":"South Texas ICE Processing Center","2":"1350","3":"734","4":"464","5":"69.891008"},{"1":"Tacoma ICE Processing Center (Northwest Det Ctr)","2":"1181","3":"311","4":"248","5":"16.398714"},{"1":"Lasalle ICE Processing Center (Jena)","2":"1170","3":"464","4":"344","5":"55.172414"},{"1":"Otay Mesa Detention Center (San Diego Cdf)","2":"1100","3":"352","4":"227","5":"53.977273"},{"1":"Adams County Det Center","2":"1100","3":"348","4":"318","5":"65.804598"},{"1":"Winn Correctional Center","2":"946","3":"365","4":"301","5":"57.260274"},{"1":"Karnes County Residential Center","2":"830","3":"112","4":"34","5":"100.000000"},{"1":"Port Isabel","2":"800","3":"392","4":"190","5":"83.163265"},{"1":"Jackson Parish Correctional Center","2":"751","3":"184","4":"144","5":"72.826087"},{"1":"Bluebonnet Detention Facility","2":"750","3":"374","4":"227","5":"31.016043"},{"1":"Montgomery ICE Processing Center","2":"750","3":"329","4":"207","5":"46.504559"},{"1":"El Valle Detention Facility","2":"750","3":"327","4":"175","5":"83.486239"},{"1":"Houston Contract Detention Facility","2":"750","3":"190","4":"124","5":"58.947368"},{"1":"Torrance County Detention Facility","2":"714","3":"24","4":"21","5":"41.666667"},{"1":"Broward Transitional Center","2":"700","3":"331","4":"193","5":"77.643505"},{"1":"South Louisiana Detention Center","2":"700","3":"84","4":"73","5":"71.428571"},{"1":"Richwood Correctional Center","2":"677","3":"156","4":"134","5":"80.128205"},{"1":"Imperial Regional Detention Facility","2":"640","3":"314","4":"203","5":"79.936306"},{"1":"El Paso Service Processing Center","2":"600","3":"323","4":"159","5":"67.801858"},{"1":"Irwin County Detention Center","2":"600","3":"314","4":"248","5":"31.847134"},{"1":"Golden State Annex","2":"560","3":"83","4":"64","5":"3.614458"},{"1":"Prairieland Detention Facility","2":"550","3":"292","4":"214","5":"31.849315"},{"1":"Folkston Main Ipc","2":"544","3":"123","4":"50","5":"39.837398"},{"1":"Denver Contract Detention Facility","2":"525","3":"218","4":"154","5":"19.724771"},{"1":"York County Prison","2":"500","3":"327","4":"250","5":"22.935780"},{"1":"Otero County Processing Center","2":"500","3":"188","4":"125","5":"55.851064"},{"1":"Immigration Centers Of America Farmville","2":"500","3":"101","4":"70","5":"15.841584"},{"1":"T. Don Hutto Detention Center","2":"461","3":"71","4":"52","5":"98.591549"},{"1":"Krome North Service Processing Center","2":"450","3":"310","4":"217","5":"30.967742"},{"1":"Denver Contract Detention Facility (Cdf) Ii","2":"432","3":"38","4":"28","5":"10.526316"},{"1":"Buffalo (Batavia) Service Processing Center","2":"400","3":"252","4":"218","5":"18.650794"},{"1":"Florence Service Processing Center","2":"392","3":"72","4":"44","5":"61.111111"},{"1":"River Correctional Center","2":"361","3":"140","4":"109","5":"80.000000"},{"1":"Iah Secure Adult Detention Facility (Polk)","2":"350","3":"76","4":"64","5":"60.526316"},{"1":"Mesa Verde ICE Processing Center","2":"320","3":"43","4":"39","5":"2.325581"},{"1":"Glades County Detention Center","2":"300","3":"257","4":"171","5":"28.015564"},{"1":"Elizabeth Contract Detention Facility","2":"285","3":"102","4":"58","5":"75.490196"},{"1":"Rio Grande Detention Center","2":"275","3":"155","4":"118","5":"69.677419"},{"1":"Caroline Detention Facility","2":"224","3":"176","4":"108","5":"19.318182"},{"1":"Yuba County Jail","2":"150","3":"19","4":"17","5":"0.000000"},{"1":"Desert View","2":"120","3":"13","4":"11","5":"15.384615"},{"1":"Allen Parish Public Safety Complex","2":"100","3":"64","4":"55","5":"84.375000"},{"1":"San Luis Regional Detention Center","2":"100","3":"54","4":"31","5":"64.814815"},{"1":"Calhoun County Correctional Center","2":"75","3":"114","4":"83","5":"18.421053"},{"1":"Northwestern Regional Juvenile Detention Center","2":"2","3":"0","4":"0","5":"NaN"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

## Estimated number of "Ghost Beds"
Selman and Leighton [-@selman2010, 114] explain that compensation for private prisons is fraught with guarantees, including the payment of a minimum number of inmates which over time became standard language in many contracts. Selman and Leighton refer to the payment of nonexistent inmate beds through guaranteed minimums as "ghost inmates." Immigration detention is modeled on corrections and is effectively punishment imposed  through a program of prevention through deterrence. However, individuals incarcerated by ICE are held in what is supposed to be non-punitive civil custody. Therefore, rather than referring to empty guaranteed beds that are paid for through contractual obligations as "ghost inmates" I refer to these in the immigration context as "ghost beds."

```r
ghost_beds <- facilities %>% 
  select(c(Name, `Guaranteed Minimum`, `Total Detained`, `Percent No Threat`)) %>% 
  filter(`Guaranteed Minimum` >0) %>% 
  mutate(`Estimated Ghost Beds` = `Guaranteed Minimum` - `Total Detained`) %>% 
  select(c(Name, `Estimated Ghost Beds`)) %>% 
  arrange(desc(`Estimated Ghost Beds`))
```




Based on ICE data, for FY21 there are a total of 22960 ghost beds. Assuming an average cost of $100 per day per bed, this comes out to \$2296000 per day paid for empty detention beds.


```r
as_tibble(ghost_beds) %>% 
  select(-geometry)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Name"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Estimated Ghost Beds"],"name":[2],"type":["dbl"],"align":["right"]}],"data":[{"1":"South Texas Family Residential Center","2":"2101"},{"1":"La Palma Correction Center - Apso","2":"1422"},{"1":"La Palma Correctional Center","2":"1372"},{"1":"Adelanto ICE Processing Center","2":"1119"},{"1":"Stewart Detention Center","2":"927"},{"1":"Tacoma ICE Processing Center (Northwest Det Ctr)","2":"870"},{"1":"Adams County Det Center","2":"752"},{"1":"Otay Mesa Detention Center (San Diego Cdf)","2":"748"},{"1":"Karnes County Residential Center","2":"718"},{"1":"Lasalle ICE Processing Center (Jena)","2":"706"},{"1":"Torrance County Detention Facility","2":"690"},{"1":"South Texas ICE Processing Center","2":"616"},{"1":"South Louisiana Detention Center","2":"616"},{"1":"Winn Correctional Center","2":"581"},{"1":"Jackson Parish Correctional Center","2":"567"},{"1":"Houston Contract Detention Facility","2":"560"},{"1":"Richwood Correctional Center","2":"521"},{"1":"Golden State Annex","2":"477"},{"1":"El Valle Detention Facility","2":"423"},{"1":"Montgomery ICE Processing Center","2":"421"},{"1":"Folkston Main Ipc","2":"421"},{"1":"Port Isabel","2":"408"},{"1":"Immigration Centers Of America Farmville","2":"399"},{"1":"Denver Contract Detention Facility (Cdf) Ii","2":"394"},{"1":"T. Don Hutto Detention Center","2":"390"},{"1":"Bluebonnet Detention Facility","2":"376"},{"1":"Broward Transitional Center","2":"369"},{"1":"Imperial Regional Detention Facility","2":"326"},{"1":"Florence Service Processing Center","2":"320"},{"1":"Otero County Processing Center","2":"312"},{"1":"Denver Contract Detention Facility","2":"307"},{"1":"Irwin County Detention Center","2":"286"},{"1":"El Paso Service Processing Center","2":"277"},{"1":"Mesa Verde ICE Processing Center","2":"277"},{"1":"Iah Secure Adult Detention Facility (Polk)","2":"274"},{"1":"Prairieland Detention Facility","2":"258"},{"1":"River Correctional Center","2":"221"},{"1":"Elizabeth Contract Detention Facility","2":"183"},{"1":"York County Prison","2":"173"},{"1":"Buffalo (Batavia) Service Processing Center","2":"148"},{"1":"Krome North Service Processing Center","2":"140"},{"1":"Yuba County Jail","2":"131"},{"1":"Rio Grande Detention Center","2":"120"},{"1":"Desert View","2":"107"},{"1":"Caroline Detention Facility","2":"48"},{"1":"San Luis Regional Detention Center","2":"46"},{"1":"Glades County Detention Center","2":"43"},{"1":"Allen Parish Public Safety Complex","2":"36"},{"1":"Northwestern Regional Juvenile Detention Center","2":"2"},{"1":"Calhoun County Correctional Center","2":"-39"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>



```r
ggplot()+
  geom_sf(data=states)+
  geom_sf(data=ghost_beds, aes(size=`Estimated Ghost Beds`))+
  coord_sf(xlim = c(-125,-64), ylim = c(16.5, 49.5), expand = FALSE)+
    labs(
    title = "ICE Detention Facilities",
    subtitle = "Showing Estimated Ghost Beds FY21",
    caption = "Data source: https://www.ice.gov/detain/detention-management/",
    size = "Estimated\nGhost Beds")
```

![](facility_plots_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

## Threat Level


```r
no_threat <- 
  facilities %>% 
  select(c(Name, `Percent No Threat`, `Total Detained`)) %>% 
  arrange(desc(`Percent No Threat`))
as_tibble(no_threat) %>% 
  select(-geometry)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Name"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Percent No Threat"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["Total Detained"],"name":[3],"type":["dbl"],"align":["right"]}],"data":[{"1":"Karnes County Residential Center","2":"100.000000","3":"112"},{"1":"Cbp San Ysidro Poe","2":"100.000000","3":"2"},{"1":"Madison County Jail","2":"100.000000","3":"1"},{"1":"South Texas Family Residential Center","2":"99.331104","3":"299"},{"1":"T. Don Hutto Detention Center","2":"98.591549","3":"71"},{"1":"Cca, Florence Correctional Center","2":"85.714286","3":"84"},{"1":"Cbp Chula Vista Bps","2":"84.615385","3":"13"},{"1":"Allen Parish Public Safety Complex","2":"84.375000","3":"64"},{"1":"El Valle Detention Facility","2":"83.486239","3":"327"},{"1":"Clinton County Jail","2":"83.333333","3":"6"},{"1":"Port Isabel","2":"83.163265","3":"392"},{"1":"Webb County Detention Center (Cca)","2":"81.395349","3":"86"},{"1":"Richwood Correctional Center","2":"80.128205","3":"156"},{"1":"River Correctional Center","2":"80.000000","3":"140"},{"1":"Imperial Regional Detention Facility","2":"79.936306","3":"314"},{"1":"Broward Transitional Center","2":"77.643505","3":"331"},{"1":"La Palma Correctional Center","2":"77.570093","3":"428"},{"1":"Limestone County Detention Center","2":"76.712329","3":"146"},{"1":"Eloy Federal Contract Facility","2":"76.640420","3":"381"},{"1":"Laredo Processing Center","2":"75.789474","3":"95"},{"1":"Elizabeth Contract Detention Facility","2":"75.490196","3":"102"},{"1":"Jackson Parish Correctional Center","2":"72.826087","3":"184"},{"1":"South Louisiana Detention Center","2":"71.428571","3":"84"},{"1":"South Texas ICE Processing Center","2":"69.891008","3":"734"},{"1":"Rio Grande Detention Center","2":"69.677419","3":"155"},{"1":"La Palma Correction Center - Apso","2":"69.312169","3":"378"},{"1":"El Paso Service Processing Center","2":"67.801858","3":"323"},{"1":"Adams County Det Center","2":"65.804598","3":"348"},{"1":"San Luis Regional Detention Center","2":"64.814815","3":"54"},{"1":"Florence Service Processing Center","2":"61.111111","3":"72"},{"1":"Iah Secure Adult Detention Facility (Polk)","2":"60.526316","3":"76"},{"1":"Houston Contract Detention Facility","2":"58.947368","3":"190"},{"1":"Winn Correctional Center","2":"57.260274","3":"365"},{"1":"Joe Corley Processing Ctr","2":"56.250000","3":"32"},{"1":"Otero County Processing Center","2":"55.851064","3":"188"},{"1":"Pine Prairie ICE Processing Center","2":"55.299539","3":"217"},{"1":"Lasalle ICE Processing Center (Jena)","2":"55.172414","3":"464"},{"1":"Otay Mesa Detention Center (San Diego Cdf)","2":"53.977273","3":"352"},{"1":"Etowah County Jail (Alabama)","2":"50.515464","3":"97"},{"1":"Coastal Bend Detention Facility","2":"50.000000","3":"4"},{"1":"Garvin County Detention Center","2":"50.000000","3":"2"},{"1":"Montgomery ICE Processing Center","2":"46.504559","3":"329"},{"1":"Florence Staging Facility","2":"45.945946","3":"37"},{"1":"Clay County Jail","2":"45.098039","3":"51"},{"1":"Strafford County Corrections","2":"45.070423","3":"71"},{"1":"Euless City Jail","2":"42.857143","3":"7"},{"1":"Torrance County Detention Facility","2":"41.666667","3":"24"},{"1":"Plymouth County Correctional Facility","2":"40.740741","3":"81"},{"1":"Folkston Main Ipc","2":"39.837398","3":"123"},{"1":"Okmulgee County Jail","2":"39.240506","3":"79"},{"1":"Rolling Plains Detention Center","2":"38.888889","3":"36"},{"1":"Kay County Justice Facility","2":"38.775510","3":"49"},{"1":"Monroe County Detention-Dorm","2":"37.500000","3":"8"},{"1":"Essex County Correctional Facility","2":"37.320574","3":"209"},{"1":"Johnson County Corrections Center","2":"36.363636","3":"33"},{"1":"Alexandria Staging Facility","2":"36.000000","3":"125"},{"1":"Mchenry County Correctional Facility","2":"33.333333","3":"108"},{"1":"Orange County Jail","2":"33.333333","3":"69"},{"1":"Saint Clair County Jail","2":"33.333333","3":"27"},{"1":"Seneca County Jail","2":"33.333333","3":"27"},{"1":"Collier County Naples Jail Center","2":"33.333333","3":"3"},{"1":"Pike County Correctional Facility","2":"32.558140","3":"43"},{"1":"Wyatt Detention Center","2":"32.258065","3":"31"},{"1":"Prairieland Detention Facility","2":"31.849315","3":"292"},{"1":"Irwin County Detention Center","2":"31.847134","3":"314"},{"1":"Bluebonnet Detention Facility","2":"31.016043","3":"374"},{"1":"Krome North Service Processing Center","2":"30.967742","3":"310"},{"1":"Stewart Detention Center","2":"30.609212","3":"673"},{"1":"Pulaski County Jail","2":"29.885057","3":"87"},{"1":"Nevada Southern Detention Center","2":"29.761905","3":"84"},{"1":"Kankakee County Jail (Jerome Combs Det Ctr)","2":"29.729730","3":"37"},{"1":"Boone County Jail","2":"29.090909","3":"55"},{"1":"Cambria County Jail","2":"28.571429","3":"7"},{"1":"Rensselaer County Correctional Facility","2":"28.571429","3":"7"},{"1":"Glades County Detention Center","2":"28.015564","3":"257"},{"1":"Bergen County Jail","2":"27.950311","3":"161"},{"1":"Baker County Sheriff'S Office","2":"27.160494","3":"162"},{"1":"Henderson Detention Center","2":"26.612903","3":"124"},{"1":"Eden Detention Center","2":"26.315789","3":"95"},{"1":"Polk County Jail","2":"25.925926","3":"27"},{"1":"Butler County Jail","2":"25.000000","3":"92"},{"1":"Geauga County Jail","2":"25.000000","3":"24"},{"1":"Chase County Detention Facility","2":"24.000000","3":"50"},{"1":"Worcester County Jail","2":"23.809524","3":"21"},{"1":"Robert A. Deyton Detention Facility","2":"23.529412","3":"17"},{"1":"York County Prison","2":"22.935780","3":"327"},{"1":"Franklin County House Of Correction","2":"22.727273","3":"22"},{"1":"Hudson County Correctional Center","2":"22.535211","3":"71"},{"1":"Freeborn County Adult Detention Center","2":"22.448980","3":"49"},{"1":"Sherburne County Jail","2":"21.538462","3":"65"},{"1":"Kandiyohi County Jail","2":"20.000000","3":"70"},{"1":"Bristol County Detention Center","2":"20.000000","3":"20"},{"1":"Pottawattamie County Jail","2":"20.000000","3":"5"},{"1":"Teller County Jail","2":"20.000000","3":"5"},{"1":"Denver Contract Detention Facility","2":"19.724771","3":"218"},{"1":"Nye County Detention Center, Southern (Pahrump)","2":"19.696970","3":"66"},{"1":"Caroline Detention Facility","2":"19.318182","3":"176"},{"1":"Buffalo (Batavia) Service Processing Center","2":"18.650794","3":"252"},{"1":"Calhoun County Correctional Center","2":"18.421053","3":"114"},{"1":"Annex - Folkston Ipc","2":"18.421053","3":"38"},{"1":"Wakulla County Jail","2":"16.666667","3":"48"},{"1":"Tacoma ICE Processing Center (Northwest Det Ctr)","2":"16.398714","3":"311"},{"1":"Immigration Centers Of America Farmville","2":"15.841584","3":"101"},{"1":"Dodge County Jail","2":"15.714286","3":"70"},{"1":"Desert View","2":"15.384615","3":"13"},{"1":"Adelanto ICE Processing Center","2":"12.797619","3":"336"},{"1":"Phelps County Jail","2":"12.500000","3":"16"},{"1":"Washoe County Jail","2":"11.111111","3":"9"},{"1":"Chippewa County Ssm","2":"11.111111","3":"9"},{"1":"Hardin County Jail","2":"10.810811","3":"37"},{"1":"Denver Contract Detention Facility (Cdf) Ii","2":"10.526316","3":"38"},{"1":"Hall County Department Of Corrections","2":"9.523810","3":"21"},{"1":"Howard County Detention Center","2":"7.142857","3":"14"},{"1":"Douglas County Department Of Corrections","2":"6.250000","3":"16"},{"1":"Honolulu Federal Detention Center","2":"5.882353","3":"17"},{"1":"Golden State Annex","2":"3.614458","3":"83"},{"1":"Clinton County Correctional Facility","2":"2.469136","3":"81"},{"1":"Mesa Verde ICE Processing Center","2":"2.325581","3":"43"},{"1":"Department Of Corrections Hagatna","2":"0.000000","3":"22"},{"1":"Yuba County Jail","2":"0.000000","3":"19"},{"1":"Saipan Department Of Corrections (Susupe)","2":"0.000000","3":"4"},{"1":"Carver County Jail","2":"0.000000","3":"4"},{"1":"Pickens County Det Ctr","2":"0.000000","3":"3"},{"1":"Guaynabo Mdc (San Juan)","2":"0.000000","3":"3"},{"1":"South Central Regional Jail","2":"0.000000","3":"2"},{"1":"Linn County Jail","2":"0.000000","3":"2"},{"1":"East Hidalgo Detention Center","2":"0.000000","3":"2"},{"1":"Pinellas County Jail","2":"0.000000","3":"1"},{"1":"Fayette County Detention Center","2":"0.000000","3":"1"},{"1":"Sweetwater County Jail","2":"0.000000","3":"1"},{"1":"Western Tennessee Detention Facility","2":"0.000000","3":"1"},{"1":"La Paz County Adult Detention Facility","2":"NaN","3":"0"},{"1":"Erie County Jail","2":"NaN","3":"0"},{"1":"Holiday Inn Express & Suites","2":"NaN","3":"0"},{"1":"Grand Forks County Correctional Facility","2":"NaN","3":"0"},{"1":"Rockingham County Jail","2":"NaN","3":"0"},{"1":"Miller County Jail","2":"NaN","3":"0"},{"1":"Dorchester County Detention Center","2":"NaN","3":"0"},{"1":"Holiday Inn Express & Suites El Paso","2":"NaN","3":"0"},{"1":"Rio Grande County Jail","2":"NaN","3":"0"},{"1":"Hiland Mountain Correctional Center","2":"NaN","3":"0"},{"1":"Northwestern Regional Juvenile Detention Center","2":"NaN","3":"0"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>


```r
no_threat %>% 
ggplot()+
  geom_histogram(aes(`Percent No Threat`), bins = 20)
```

![](facility_plots_files/figure-html/histogram no threat-1.png)<!-- -->



```r
ggplot()+
  geom_sf(data=states)+
  geom_sf(data=`no_threat`, aes(size=`Percent No Threat`, color=`Total Detained`))+
  coord_sf(xlim = c(-125,-64), ylim = c(16.5, 49.5), expand = FALSE)+
    labs(
    title = "ICE Detention Facilities",
    subtitle = "Showing Estimated Percent No ICE Threat",
    caption = "Data source: https://www.ice.gov/detain/detention-management/",
    color = "Total\nDetained",
    size = "Estimated\n% No Threat")
```

```
Warning: Removed 11 rows containing missing values (geom_sf).
```

![](facility_plots_files/figure-html/map estimated percent no threat-1.png)<!-- -->



## Mandatory Detention


```r
ggplot()+
  geom_sf(data=states)+
  geom_sf(data=facilities, aes(color = `Guaranteed Minimum`, size = `Mandatory Detention`))+
  coord_sf(xlim = c(-125,-64), ylim = c(16.5, 49.5), expand = FALSE)+
  labs(
    title = "ICE Detention Facilities",
    subtitle = "Showing Mandatory Detention and Contractual Guaranteed Minimums",
    caption = "Data source: https://www.ice.gov/detain/detention-management",
    color ="Guaranteed\nMinimum",
    size = "Mandatory\nDetention")
```

![](facility_plots_files/figure-html/facility map with mandatory detention and guaranteed minimums-1.png)<!-- -->

# References
