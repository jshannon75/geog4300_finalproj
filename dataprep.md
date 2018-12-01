Geog6300: Final project data prep
================
Jerry Shannon

``` r
library(tidycensus)
library(tidyverse)
library(sf)
library(FedData)
```

This script downloads and cleans data on the city of Atlanta for the final project in Geog6300: Data Science in Geography (fall 2018).

We can

We will start by loading a list of census variables relevant to this project. We'll download the normalizing variables--mostly total population counts--separately and then join those in.

``` r
vars<-read_csv("data/census_vars.csv")

vars_lookup<-vars %>%
  select(variable,normal_var)

vars_name<-vars %>%
  select(variable,name) 

acsdata_normal<-get_acs(geography="tract",state="GA",variables = vars$normal_var) %>%
  select(GEOID,variable,estimate) %>%
  rename("normal_var"=variable, "normest"=estimate) %>%
  distinct()
```

    ## Warning: package 'bindrcpp' was built under R version 3.4.4

``` r
acsdata<-get_acs(geography="tract",state="GA",variables = vars$variable) %>%
  left_join(vars_lookup)%>%
  left_join(acsdata_normal)
```

Next, we'll create percentages for most of these variables. We'll separate out those that don't need to be normalized--total population or median income. Then we'll combine them using bind\_cols.

``` r
acsdata_pct<-acsdata %>%
  mutate(est_pct=round(estimate/normest*100,1))

acs_data_pct1<-acsdata_pct %>%
  filter(variable!=normal_var) %>%
  left_join(vars_name) %>%
  distinct() %>%
  select(GEOID,name,est_pct) %>%
  spread(name,est_pct)


acs_data_pct2<-acsdata_pct %>%
  filter(variable==normal_var) %>%
  select(GEOID,variable,estimate) %>%
  left_join(vars_name) %>%
  select(-variable) %>%
  spread(name,estimate)

acs_data_all<-bind_cols(acs_data_pct2,acs_data_pct1)
acs_data_all<-bind_cols(select(acs_data_all,GEOID),acs_data_all[,vars$name])

#write_csv(acs_data_all,"data/acs_data_all.csv")
```

We can also read in elevation and % developed from the NED and NLCD respectively.

``` r
tracts_data<-st_read("data/atl_data.gpkg",layer="tracts_atl_2016") %>%
  select(GEOID10,SHAPE_AREA,TRACTID,GISJN_TCT,X_mean,X_count,X_sum) %>%
  mutate(develop_pct=round(X_sum/X_count*100,2)) %>%
  rename("elev_m"=X_mean) %>%
  select(-X_sum,-X_count) 
```

    ## Reading layer `tracts_atl_2016' from data source `C:\Users\jshannon\Dropbox\Jschool\Teaching\Courses\geog4300_6300_Fall18\geog4300_labs\geog4300_finalproj\data\atl_data.gpkg' using driver `GPKG'
    ## Simple feature collection with 955 features and 15 fields
    ## geometry type:  MULTIPOLYGON
    ## dimension:      XY
    ## bbox:           xmin: -85.38658 ymin: 32.84464 xmax: -83.2692 ymax: 34.61792
    ## epsg (SRID):    4326
    ## proj4string:    +proj=longlat +datum=WGS84 +no_defs

We will also download daily climate data for tract centroids. This is done using the batch extractor provided by Daymet and then those files are read into R. We summarise precipitation and temperature for 2005, 2010, and 2015.

``` r
centers<-st_centroid(tracts_data)
```

    ## Warning in st_centroid.sf(tracts_data): st_centroid assumes attributes are
    ## constant over geometries of x

    ## Warning in st_centroid.sfc(st_geometry(x), of_largest_polygon =
    ## of_largest_polygon): st_centroid does not give correct centroids for
    ## longitude/latitude data

``` r
xy<-data.frame(st_coordinates(centers))
centroids_xy<-bind_cols(xy,centers) %>%
  select(X,Y,GEOID10,TRACTID,GISJN_TCT) %>%
  mutate(coord_query=paste(GISJN_TCT,".csv, ",Y,", ",X,sep=""))
write_csv(centroids_xy,"data/centroids.csv")

daymet_files<-list.files(path="data/daymet_spt_data_extraction_v2/",pattern=(".csv"))
read_daymet<-function(file){
  filename<-paste("data/daymet_spt_data_extraction_v2/",file,sep="")
  read_csv(filename,skip=7)
}

daymet_data<-data_frame(filename=daymet_files) %>%
  mutate(file_contents=map(filename,read_daymet))  
daymet_data1<-unnest(daymet_data) %>%
  mutate(gisjn_tct=toupper(substr(filename,1,12))) %>%
  rename("prcp"=`prcp (mm/day)`,
         "tmax"=`tmax (deg c)`,
         "tmin"=`tmin (deg c)`) %>%
  select(gisjn_tct,year,yday,prcp,tmax,tmin)
daymet_summary<-daymet_data1 %>%
  group_by(gisjn_tct,year) %>%
  summarise(prcp=sum(prcp),
            tmax=mean(tmax),
            tmin=mean(tmin)) %>%
  ungroup() %>%
  group_by(gisjn_tct) %>%
  summarise(prcp=mean(prcp),
            tmax=mean(tmax),
            tmin=mean(tmin)) %>%
  rename("GISJN_TCT"=gisjn_tct)

write_csv(daymet_summary,"data/daymet_data.csv")
```

Let's join the daymet data to the tracts

``` r
tracts_data1<-left_join(tracts_data,daymet_summary) %>%
  mutate(GEOID10=as.character(GEOID10))
```

    ## Warning: Column `GISJN_TCT` joining factor and character vector, coercing
    ## into character vector

We can also read in data from the [Mixed Metro project](http://mixedmetro.com/) and add county identifiers.

``` r
mm_classes<-read_csv("data/mm_classes.csv")
tracts_mm<-read_csv("data/ustracts_mm_acs16.csv") %>%
  mutate(GEOID10=as.character(GEOID)) %>% 
  select(GEOID10,class) %>%
  rename(class_mm=class) %>%
  left_join(mm_classes)
tracts_data1a<-tracts_data1 %>%
  left_join(tracts_mm) %>%
  mutate(cty_fips=substr(GEOID10,1,5))

#Reorder the variables
tracts_data1b<-tracts_data1a %>%
  select(GEOID10,TRACTID,GISJN_TCT,cty_fips,class_mm,class_mm_char,everything()) %>%
  select(-SHAPE_AREA)
```

And also connect the ACS data. We will also filter the records so only counties with 15 or more tracts are included.

``` r
acs_data_all1<-acs_data_all %>%
  mutate("GEOID10"=as.character(GEOID)) 
tracts_data2<-left_join(tracts_data1b,acs_data_all1) %>%
  select(-GEOID)

acs_data_count<-tracts_data2 %>%
  st_set_geometry(NULL) %>%
  count(cty_fips) %>%
  filter(n>10) %>%
  select(-n)

acs_data_all1<-tracts_data2 %>%
  right_join(acs_data_count) %>%
  rename(geometry=geom)
```

Lastly, we can write the data.

``` r
ggplot(acs_data_all1) + geom_sf()
```

![](dataprep_files/figure-markdown_github/unnamed-chunk-8-1.png)

``` r
st_write(acs_data_all1,"data/atl_data.gpkg",layer="acsdata_atl_climate",delete_layer = TRUE)
```

    ## Deleting layer `acsdata_atl_climate' using driver `GPKG'
    ## Updating layer `acsdata_atl_climate' to data source `data/atl_data.gpkg' using driver `GPKG'
    ## features:       912
    ## fields:         46
    ## geometry type:  Multi Polygon

``` r
acs_data_all1_df<-acs_data_all1 %>%
  st_set_geometry(NULL)
write_csv(acs_data_all1_df,"data/atl_data.csv")
```