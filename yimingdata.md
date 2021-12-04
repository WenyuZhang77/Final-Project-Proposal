program
================
Yiming Li
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

``` r
NYPD_clean = 
  read.csv("sample.csv") %>% 
  janitor::clean_names() %>% 
  select(-x, -cmplnt_num, -cmplnt_to_dt, -cmplnt_to_tm, -rpt_dt, -x_coord_cd, 
         -y_coord_cd, -housing_psa, -patrol_boro, -loc_of_occur_desc) %>% 
  separate(cmplnt_fr_dt, into = c("month", "day", "year"), sep = "/") %>% 
  filter(!is.na(year)) %>% 
  rename(exact_crime_time = cmplnt_fr_tm, 
         precinct_of_crime = addr_pct_cd,
         description_of_crime = ofns_desc,
         completed_or_not = crm_atpt_cptd_cd,
         level_of_offense = law_cat_cd,
         name_of_borough = boro_nm,
         description_of_premises = prem_typ_desc
         )
```

    ## Warning: Expected 3 pieces. Missing pieces filled with `NA` in 9 rows [1475,
    ## 6564, 17447, 27209, 80282, 83451, 88511, 95042, 95441].

``` r
NYPD_clean %>% 
  select(completed_or_not, name_of_borough) %>% 
  group_by(completed_or_not, name_of_borough) %>% 
  summarise(count = n()) %>% 
  filter(name_of_borough != "") %>% 
  pivot_wider(names_from = "name_of_borough", 
              values_from = "count") %>% 
  knitr::kable()
```

    ## `summarise()` has grouped output by 'completed_or_not'. You can override using the `.groups` argument.

| completed\_or\_not | BRONX | BROOKLYN | MANHATTAN | QUEENS | STATEN ISLAND |
|:-------------------|------:|---------:|----------:|-------:|--------------:|
| ATTEMPTED          |   318 |      521 |       439 |    331 |            55 |
| COMPLETED          | 21304 |    29098 |     23518 |  19608 |          4669 |
