# Geog6300: Final project

This repo contains data for your final takehome project. For this project, you'll draw from a variety of variable at census tract level in the Atlanta metro area--counties with 10 or more census variables. Using the project document as a template, you'll ask a research question that uses variables from this dataset and use both descriptive and inferential statistics to answer that question. 

The script used to create this data is available on this repo--see the dataprep document. There's both geopackage and csv versions of the data, which you can access through the assignment. That assignment is listed as ....

There's multiple variables that you can use for this project. Here's the list:

* GEOID10: Tract FIPS ID (character version)
* TRACTID: Tract FIPS ID (numeric version)
* GISJN_TCT: Tract FIPS ID (for NHGIS)
* cty_fips: County FIPS ID
* class_mm: [Mixed Metro](http://mixedmetro.com) code classification
* class_mm_char: [Mixed Metro](http://mixedmetro.com) code description
* elev_m: Elevation of the tract centroid in meters
* develop_pct: % of the tract classified as developed in the [NLCD](https://catalog.data.gov/dataset/national-land-cover-database-nlcd-land-cover-collection)
* prcp: Annual precipitation at tract centroid from Daymet
* tmax: Annual mean for maximum temperature from Daymet
* tmin: Annual mean for minimum temperature from Daymet
* totpop: Total population, ACS 2012-2016
* white_nh: % classified as White, non-Hispanic, ACS 2012-2016
* afam_nh: % classified as African-American, non-Hispanic, ACS 2012-2016
* natam_nh: % classified as Native American, non-Hispanic, ACS 2012-2016
* asian_nh: % classified as Asian American, non-Hispanic, ACS 2012-2016
* hisp: % classified as Hispanic, ACS 2012-2016
* fb_pop: % population who is foreign born, ACS 2012-2016
* nonga_pop: % population born in U.S., but not in Georgia, ACS 2012-2016
* ga_pop: % population born in Georgia, ACS 2012-2016
* cmute_car: % commuting alone by car, ACS 2012-2016
* cmute_carp: % commuting in carpool, ACS 2012-2016
* cmute_public: % commuting on public transportation, ACS 2012-2016
* cmute_bike: % commuting by bicycle, ACS 2012-2016
* cmute_walk: % commuting by walking, ACS 2012-2016
* cmute_home: % working at home, ACS 2012-2016
* medinc: Tract median household income
* povpop_50pct: % of population with hh income <50% poverty, ACS 2012-2016
* popop_99pct: % of population with hh income >50% but less than 99% poverty, ACS 2012-2016
* inc_10K: % hh with incomes <$10,000/year
* inc_150K: % hh with incomes between $150,000-$199,999/year
* inc_200K: % hh with incomes >$200,000/year
* tenure_own: % of owner occupied homes
* tenure_rent: % of renter occupied homes
* ind_ag: % employed in agriculture, forestry, fishing, hunting, and mining
* ind_const: % employed in construction
* ind_manuf: % employed in manufacturing
* ind_wholetrd: % employed in wholesale trade
* ind_retltrd: % employed in retail trade
* ind_transp: % employed in transportation and warehousing, utilities
* ind_info: % employed in information
* ind_finance: % employed in finance and insurance, real estate
* ind_prof: % employed in professional, scientific and management, and administrative
* ind_ed: % employed in education services, health care, and social assistance
* ind_arts: % employed in arts, entertainment and recreating, and food services
* ind_public: % employed in public administration