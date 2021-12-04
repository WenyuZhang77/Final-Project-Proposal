Aw-version
================
Shiwei Chen
11/25/2021

``` r
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.1 ──

    ## ✓ ggplot2 3.3.5     ✓ purrr   0.3.4
    ## ✓ tibble  3.1.4     ✓ dplyr   1.0.7
    ## ✓ tidyr   1.1.3     ✓ stringr 1.4.0
    ## ✓ readr   2.0.1     ✓ forcats 0.5.1

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(rvest)
```

    ## 
    ## Attaching package: 'rvest'

    ## The following object is masked from 'package:readr':
    ## 
    ##     guess_encoding

``` r
library(httr)
```

read data

``` r
NYPD = 
  GET("https://data.cityofnewyork.us/resource/qgea-i56i.csv", 
      query = list("$limit" = 250000)) %>% 
  content("parsed")
```

    ## Rows: 250000 Columns: 35

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (19): ofns_desc, pd_desc, crm_atpt_cptd_cd, law_cat_cd, boro_nm, loc_of...
    ## dbl  (11): cmplnt_num, addr_pct_cd, ky_cd, pd_cd, jurisdiction_code, housing...
    ## dttm  (3): cmplnt_fr_dt, cmplnt_to_dt, rpt_dt
    ## time  (2): cmplnt_fr_tm, cmplnt_to_tm

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
NYPD_clean = 
  NYPD %>% 
  janitor::clean_names() %>% 
  select(-cmplnt_num, -cmplnt_to_dt, -cmplnt_to_tm, -rpt_dt, -x_coord_cd, -y_coord_cd, -housing_psa, -patrol_boro, -loc_of_occur_desc) %>% 
  separate(cmplnt_fr_dt, into = c("year", "month", "day"), sep = "-") %>% 
    rename(exact_crime_time = cmplnt_fr_tm, 
         precinct_of_crime = addr_pct_cd,
         description_of_crime = ofns_desc,
         completed_or_not = crm_atpt_cptd_cd,
         level_of_offense = law_cat_cd,
         name_of_borough = boro_nm,
         description_of_premises = prem_typ_desc
         )
```

``` r
NYPD_race = NYPD_clean %>% 
  select(vic_race, vic_age_group) %>% 
  group_by(vic_race) %>% 
  summarize(number_of_people = n()) %>% 
  mutate(vic_race = reorder(vic_race, number_of_people)) %>% 
  knitr::kable()
NYPD_race
```

| vic\_race                      | number\_of\_people |
|:-------------------------------|-------------------:|
| AMERICAN INDIAN/ALASKAN NATIVE |               1508 |
| ASIAN / PACIFIC ISLANDER       |              19338 |
| BLACK                          |              63106 |
| BLACK HISPANIC                 |              10552 |
| UNKNOWN                        |              72313 |
| WHITE                          |              39954 |
| WHITE HISPANIC                 |              43226 |
| NA                             |                  3 |
