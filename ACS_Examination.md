---
title: "ACS Census Data"
author: "Andrea Bonilla"
output: 
  html_document:
    keep_md: true
---

Code to examine the ACS Census data, which contains statistics about gender and pay as well. This will be based on the tidycensus package (https://github.com/GL-Li/totalcensus)

## Set Up Our Code

We need to load in the correct packages, and set up an API key and a path where we want our data to be loaded to. The Census API offers an easy way to download data. You need to get a key from this website, http://api.census.gov/data/key_signup.html. Once you have a key, you can replace the text and use replace "YOUR API KEY GOES HERE"



We want to examine the different options associated with the ACS data. Feel free to filter the different datasets below to generate different data.


```r
vars <- search_tablecontents("acs", view=FALSE) #figure out which variables you want
summary_levs <- search_summarylevels("acs", view=FALSE) # figure out which summary levels we have access to
geos <- search_geocomponents("acs", view=FALSE)
geos2 <- search_geoheaders("acs", view=FALSE)
```

### DMV Map

We want to create the first base map of the DMV, or DC-Maryland-Virginia area. We'll use county level data.  


```r
#from the tigris package we can get each individual county from state
states <- c("DC","VA","MD")
dmv <- rbind_tigris(
  lapply(
    states, function(x) counties(state = x,cb =TRUE)
  )
)
```

![](ACS_Examination_files/figure-html/dmv-map-1.png)<!-- -->




### Visualize the Gender Ratio in the DMV.

We want to visualize the gender ration in each area. We'll create a function 'load_data_fm' to get ACS 5 year county estimates for males and females, for each state.  


```r
load_data_fm <- function(state){
  return(get_acs("county", variables=c(male="B01001_002",female="B01001_026"), state=state, survey="acs5"))
}

dc_fm <- load_data_fm("DC")
```

```
## Please note: `get_acs()` now defaults to a year or endyear of 2016.
```

```r
md_fm <- load_data_fm("MD")
```

```
## Please note: `get_acs()` now defaults to a year or endyear of 2016.
```

```r
va_fm <- load_data_fm("VA")
```

```
## Please note: `get_acs()` now defaults to a year or endyear of 2016.
```

```r
dmv_fm <- rbind(dc_fm, md_fm, va_fm) #combine each state together
dmv_fm <- dmv_fm[,-5]

dmv_fm2 <- dmv_fm %>% spread(variable, estimate) #transpose the male and female variables
dmv_fm2$pfemale <- round(dmv_fm2$female / (dmv_fm2$female + dmv_fm2$male) * 100 , 0) #calculate the female percentage

print(head(dmv_fm2))
```

```
## # A tibble: 6 x 5
##   GEOID NAME                                       female   male pfemale
##   <chr> <chr>                                       <dbl>  <dbl>   <dbl>
## 1 11001 District of Columbia, District of Columbia 346380 312629    53.0
## 2 24001 Allegany County, Maryland                   34983  38077    48.0
## 3 24003 Anne Arundel County, Maryland              282895 276842    51.0
## 4 24005 Baltimore County, Maryland                 434564 391102    53.0
## 5 24009 Calvert County, Maryland                    45561  44966    50.0
## 6 24011 Caroline County, Maryland                   16831  15822    52.0
```

Now we can map the percentage female in each county. 


```r
dmv2 <- geo_join(dmv, dmv_fm2, "GEOID", "GEOID", how="inner") #158 rows

pal <- colorQuantile("Blues", NULL, n = 3)

map1 <- leaflet() %>%
  addProviderTiles("CartoDB.Positron") %>%
  addPolygons(data = dmv2, 
              fillColor = ~pal(dmv2$pfemale), 
              fillOpacity = 0.7, 
              weight = 0.2, 
              smoothFactor = 0.2, 
              popup = paste("Percent Female: ", dmv2$pfemale, "%", sep="" )) %>%
  addLegend(pal = pal, 
            values = dmv2$pfemale, 
            position = "bottomright", 
            title = "Percent Female")

map1
```

<!--html_preserve--><div id="htmlwidget-da113b378235fe98edfa" style="width:672px;height:480px;" class="leaflet html-widget"></div>

### Gender Inequality Statistics

We want to filter for the counts of individuals in each category. The table_contents variable is how we do that. m1_2499 translates to Male $1-$2,499 income level count. 


```r
load_data_pay <- function(state){
  return(get_acs("county", variables=c(m1_2499 = "B20001_003", m2500_4999 = "B20001_004",
                                       m5000_7499 ="B20001_005",
                       m7500_9999 = "B20001_006",m10000_12499 = "B20001_007",
                       m12500_14999 = "B20001_008",m15000_17499 =  "B20001_009",
                       m17500_19999 = "B20001_010",
                       m20000_22499 = "B20001_011",
                       m22500_24999 = "B20001_012",
                      m25000_29999 = "B20001_013",
                      m30000_34999 = "B20001_014",
                       m35000_39999 = "B20001_015",
                       m40000_44999 = "B20001_016",
                       m45000_49999 = "B20001_017",
                       m50000_54999 = "B20001_018",
                       m55000_64999 = "B20001_019",
                       m65000_74999 = "B20001_020",
                       m75000_99999 = "B20001_021",
                       m100000_ ="B20001_022",
                       f1_2499 =  "B20001_024",
                       f2500_4999 = "B20001_025",
                       f5000_7499 = "B20001_026",
                       f7500_9999 = "B20001_027",
                       f10000_12499 = "B20001_028",
                       f12500_14999 = "B20001_029",
                       f15000_17499 = "B20001_030",
                       f17500_19999 = "B20001_031",
                       f20000_22499 = "B20001_032",
                       f22500_24999 = "B20001_033",
                       f25000_29999 =  "B20001_034",
                       f30000_34999 = "B20001_035",
                       f35000_39999 = "B20001_036",
                       f40000_44999 = "B20001_037",
                       f45000_49999 = "B20001_038",
                       f50000_54999 = "B20001_039",
                       f55000_64999 = "B20001_040",
                       f65000_74999 = "B20001_041",
                      f75000_99999 = "B20001_042",
                      f100000_ = "B20001_043"), state=state, survey="acs5"))
}

dc_pay <- load_data_pay("DC")
```

```
## Please note: `get_acs()` now defaults to a year or endyear of 2016.
```

```r
md_pay <- load_data_pay("MD")
```

```
## Please note: `get_acs()` now defaults to a year or endyear of 2016.
```

```r
va_pay <- load_data_pay("VA")
```

```
## Please note: `get_acs()` now defaults to a year or endyear of 2016.
```

```r
dmv_pay <- rbind(dc_pay, md_pay, va_pay)
dmv_pay <- dmv_pay[,-5]
dmv_pay2 <- dmv_pay %>% spread(variable, estimate) #transpose the male and female variables

print(head(dmv_pay2))
```

```
## # A tibble: 6 x 42
##   GEOID NAME       f1_2499 f10000_12499 f100000_ f12500_14999 f15000_17499
##   <chr> <chr>        <dbl>        <dbl>    <dbl>        <dbl>        <dbl>
## 1 11001 District ~   14355         7270    34530         3772         5645
## 2 24001 Allegany ~    1686          987      168          896          923
## 3 24003 Anne Arun~    8249         6617    16666         3768         5296
## 4 24005 Baltimore~   13828         9688    16294         5831         7135
## 5 24009 Calvert C~    1584         1228     2655          379          741
## 6 24011 Caroline ~     679          432      224          323          533
## # ... with 35 more variables: f17500_19999 <dbl>, f20000_22499 <dbl>,
## #   f22500_24999 <dbl>, f2500_4999 <dbl>, f25000_29999 <dbl>,
## #   f30000_34999 <dbl>, f35000_39999 <dbl>, f40000_44999 <dbl>,
## #   f45000_49999 <dbl>, f5000_7499 <dbl>, f50000_54999 <dbl>,
## #   f55000_64999 <dbl>, f65000_74999 <dbl>, f7500_9999 <dbl>,
## #   f75000_99999 <dbl>, m1_2499 <dbl>, m10000_12499 <dbl>, m100000_ <dbl>,
## #   m12500_14999 <dbl>, m15000_17499 <dbl>, m17500_19999 <dbl>,
## #   m20000_22499 <dbl>, m22500_24999 <dbl>, m2500_4999 <dbl>,
## #   m25000_29999 <dbl>, m30000_34999 <dbl>, m35000_39999 <dbl>,
## #   m40000_44999 <dbl>, m45000_49999 <dbl>, m5000_7499 <dbl>,
## #   m50000_54999 <dbl>, m55000_64999 <dbl>, m65000_74999 <dbl>,
## #   m7500_9999 <dbl>, m75000_99999 <dbl>
```

### Clean data now

We want to clean the above data and calculate the total sum of all female income, total sum of all male income, and then the difference between averages. 


```r
names <- names(dmv_pay2)[3:length(names(dmv_pay2))]
maths <- ifelse(names %in% c("f100000_","m100000_"), 200000 , gsub("f","", gsub("m", "", gsub("_", "+", names))))
maths2 <- sapply(maths, FUN=function(x) (eval(parse(text=x))+1)/2) # we will apply the average of each group to each column

malesum = rep(0, nrow(dmv_pay2))
femalesum = rep(0, nrow(dmv_pay2))
for(i in 1:length(names)){
  if(substr(names[i],1,1)=="f"){
    femalesum <- femalesum + dmv_pay2[[names[i]]]*as.numeric(maths2[i])
  }
  else {
    malesum <- malesum + dmv_pay2[[names[i]]]*as.numeric(maths2[i])
  }
}

dmv_pay2$malesum <- malesum
dmv_pay2$femalesum <- femalesum

## merge dmv_fm2 to get female and male totals
dmv_pay3 <- inner_join(dmv_pay2, dmv_fm2) #should be the same number of rows
```

```
## Joining, by = c("GEOID", "NAME")
```

```r
if(nrow(dmv_pay3)!= nrow(dmv_pay2)) print("ERROR CHECK OUT")

dmv_pay3$femaleavg <- round(dmv_pay3$femalesum/dmv_pay3$female,2)
dmv_pay3$maleavg <- round(dmv_pay3$malesum/dmv_pay3$male,2)

dmv_pay3$dif <- dmv_pay3$femaleavg - dmv_pay3$maleavg #difference between female and male
#positive is better

print(dmv_pay3[, c(1:2, 44:length(names(dmv_pay3)))])
```

```
## # A tibble: 158 x 9
##    GEOID NAME     femalesum female   male pfemale femaleavg maleavg    dif
##    <chr> <chr>        <dbl>  <dbl>  <dbl>   <dbl>     <dbl>   <dbl>  <dbl>
##  1 11001 Distric~    9.94e9 346380 312629    53.0     28701   33877 - 5176
##  2 24001 Allegan~    4.49e8  34983  38077    48.0     12827   16596 - 3769
##  3 24003 Anne Ar~    6.75e9 282895 276842    51.0     23877   34184 -10306
##  4 24005 Baltimo~    9.31e9 434564 391102    53.0     21420   28553 - 7133
##  5 24009 Calvert~    1.07e9  45561  44966    50.0     23410   33482 -10072
##  6 24011 Carolin~    2.46e8  16831  15822    52.0     14636   21704 - 7068
##  7 24013 Carroll~    1.77e9  84643  82892    51.0     20936   33395 -12459
##  8 24015 Cecil C~    9.14e8  51572  50603    50.0     17723   28484 -10761
##  9 24017 Charles~    2.03e9  79868  74489    52.0     25367   31596 - 6230
## 10 24019 Dorches~    2.76e8  16932  15519    52.0     16299   21184 - 4884
## # ... with 148 more rows
```

### Comparing the Pay Gap by Count

Now we will map our country differences. 


```r
dmv_pay4 <- geo_join(dmv, dmv_pay3, "GEOID", "GEOID", how="inner") #158 rows
if(nrow(dmv_pay4@data) != nrow(dmv_pay3)) print("ERROR CHECK OUT")

pal2 <- colorBin("Blues", domain=c(min(dmv_pay3$dif), max(dmv_pay3$dif)), bins = 8)

map2 <- leaflet() %>%
  addProviderTiles("CartoDB.Positron") %>%
  addPolygons(data = dmv_pay4, 
              fillColor = ~pal2(dmv_pay4$dif), 
              fillOpacity = 0.7, 
              weight = 0.2, 
              smoothFactor = 0.2, 
              popup = paste("Difference:", format(dmv_pay4$dif, big.mark=","),sep=" ")) %>%
  addLegend(pal = pal2, 
            values = dmv_pay4$dif, 
            position = "bottomright", 
            title = "Income Difference Men - Women",
            labFormat = labelFormat(prefix="$")) #%>% labFormat("bin")
```

```
## Warning in RColorBrewer::brewer.pal(max(3, n), palette): n too large, allowed maximum for palette Blues is 9
## Returning the palette you asked for with that many colors

## Warning in RColorBrewer::brewer.pal(max(3, n), palette): n too large, allowed maximum for palette Blues is 9
## Returning the palette you asked for with that many colors
```

```r
map2
```

<!--html_preserve--><div id="htmlwidget-3247ee68c87456d46b12" style="width:672px;height:480px;" class="leaflet html-widget"></div>

### Examine the Average Amount of Differences


```r
library(scales)
```

```
## 
## Attaching package: 'scales'
```

```
## The following object is masked from 'package:purrr':
## 
##     discard
```

```
## The following object is masked from 'package:readr':
## 
##     col_factor
```

```r
differences <- dmv_pay3[,c(1:2, 48:50)]
differences2 <- differences[,-5] %>% gather(gender, amount, femaleavg:maleavg, factor_key=TRUE)

diffavg <- differences2 %>% group_by(gender) %>% summarize(mean.inc = mean(amount))

ggplot(data=differences2, aes(x=amount, fill=gender,color=gender)) + geom_histogram(aes(y=..density..),alpha=.5) +
  geom_density(alpha=.2) + geom_vline(data=diffavg, aes(xintercept=mean.inc,  colour=gender),
               linetype="dashed", size=1) + ggtitle("Female and Male Average Comparisons") +
  ylab("Density") + xlab("Average Income by County") + theme_bw() + 
  scale_y_continuous(labels = scales::percent) + scale_x_continuous(labels = scales::comma)
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](ACS_Examination_files/figure-html/avgs-1.png)<!-- -->


```r
fem.higher = length(which(dmv_pay3$dif>0))
total = length(dmv_pay3$dif)

print(paste("There are only", fem.higher, "counties, out of", total, "where females make more than men"))
```

```
## [1] "There are only 2 counties, out of 158 where females make more than men"
```

```r
print(dmv_pay3[dmv_pay3$dif>0,c(2, 43:50)])
```

```
## # A tibble: 2 x 9
##   NAME      malesum femalesum female  male pfemale femaleavg maleavg   dif
##   <chr>       <dbl>     <dbl>  <dbl> <dbl>   <dbl>     <dbl>   <dbl> <dbl>
## 1 Greensvi~  8.34e7  58590066   4444  7254    38.0     13184   11491  1693
## 2 Sussex C~  7.98e7  56930090   4018  7715    34.0     14169   10345  3824
```


## Citations
D. Kahle and H. Wickham. ggmap: Spatial Visualization with ggplot2. The R Journal, 5(1),
  144-161. URL http://journal.r-project.org/archive/2013-1/kahle-wickham.pdf