Analysis of OA Data From FOCB – Revised Graphics
================
Curtis C. Bohlen, Casco Bay Estuary Partnership
7/15/2020

-   [Establish Color Palettes](#establish-color-palettes)
-   [Load Data](#load-data)
    -   [Establish Folder Reference](#establish-folder-reference)
    -   [Load The Data](#load-the-data)
        -   [Primary Data](#primary-data)
        -   [CO2SYS Results](#co2sys-results)
        -   [Merge pH (Total Scale) data into primary
            data](#merge-ph-total-scale-data-into-primary-data)
        -   [Add A Variable for the
            Season](#add-a-variable-for-the-season)
    -   [Takahashi et al. 2002
        Relationships](#takahashi-et-al-2002-relationships)
        -   [Calculations](#calculations)
    -   [Calculate Diurnal Deviations](#calculate-diurnal-deviations)
-   [Base Graphics](#base-graphics)
    -   [Constants for Axis Labels](#constants-for-axis-labels)
    -   [Seasonal Profiles](#seasonal-profiles)
        -   [Temperature Corrected pCO2](#temperature-corrected-pco2-1)
        -   [pH (Total Scale)](#ph-total-scale)
        -   [Aragonite Saturation State](#aragonite-saturation-state)
-   [Daily Patterns](#daily-patterns)
    -   [pCO<sub>2</sub>](#pco2)
        -   [Model](#model)
        -   [Predictions](#predictions)
        -   [Corrected pCO2 Graphic](#corrected-pco2-graphic)
    -   [pH](#ph)
        -   [Model](#model-1)
        -   [Predictions](#predictions-1)
        -   [pH Graph](#ph-graph)

<img
  src="https://www.cascobayestuary.org/wp-content/uploads/2014/04/logo_sm.jpg"
  style="position:absolute;top:10px;right:50px;" />

``` r
library(tidyverse)
```

    ## -- Attaching packages --------------------------------------- tidyverse 1.3.1 --

    ## v ggplot2 3.3.5     v purrr   0.3.4
    ## v tibble  3.1.6     v dplyr   1.0.7
    ## v tidyr   1.1.4     v stringr 1.4.0
    ## v readr   2.1.1     v forcats 0.5.1

    ## -- Conflicts ------------------------------------------ tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(readxl)
library(mgcv)
```

    ## Loading required package: nlme

    ## 
    ## Attaching package: 'nlme'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     collapse

    ## This is mgcv 1.8-38. For overview type 'help("mgcv-package")'.

``` r
library(lubridate)  # here, for the make_datetime() function
```

    ## 
    ## Attaching package: 'lubridate'

    ## The following objects are masked from 'package:base':
    ## 
    ##     date, intersect, setdiff, union

``` r
library(CBEPgraphics)
load_cbep_fonts()
theme_set(theme_cbep())
```

# Establish Color Palettes

Four seasons.

``` r
seasonal_colors <- c(cbep_colors()[c(1, 4, 2)], 'orange')
```

The following is used to “synchronize” colors with graphics from
CBEP/UNH acidification monitoring.

``` r
year_colors = cbep_colors()[c(2:6,1)]
```

# Load Data

## Establish Folder Reference

``` r
sibfldnm <- 'Data'
parent   <- dirname(getwd())
sibling  <- file.path(parent,sibfldnm)

fn    <- 'CMS1 Data through 2019.xlsx'
fpath <- file.path(sibling,fn)
```

## Load The Data

We need to skip the second row here, which is inconvenient largely
because the default “guess” of data contents for each column is based on
the contents of that first row of data.

A solution in an answer to this stack overflow questions
(<https://stackoverflow.com/questions/51673418/how-to-skip-the-second-row-using-readxl>)
suggests reading in the first row only to generate names, then skip the
row of names and the row of units, and read the “REAL” data. Note that I
round the timestamp on the data to the nearest hour.

The function `read_excel()` is not reading in the dates and times
consistently. It appears the data in the source data was loaded
inconsistently, with some data entered as text, some as Excel dates, and
some as Excel date-times. They all look the same in Excel, due to
formatting, but they are different “under the hood.” We reconstruct the
time from components, also provided in the s source data. The timezone
settings for these times are not 100% certain. here I read the data in
with local clock time, which I believe is usually correct for FOCB’s
data.

### Primary Data

``` r
mynames <- read_excel(fpath, sheet = 'Sheet1', n_max = 1, col_names = FALSE)
```

    ## New names:
    ## * `` -> ...1
    ## * `` -> ...2
    ## * `` -> ...3
    ## * `` -> ...4
    ## * `` -> ...5
    ## * ...

``` r
mynames <- unname(unlist(mynames[1,]))  # flatten and simplify
mynames[2] <- 'datetime'               # 
mynames[4] <- 'depth'                   # Address non-standard names
mynames[8] <- 'pctsat'
mynames[18] <- 'omega_a'
mynames <- tolower(mynames)             # convert all to lower case

the_data <- read_excel(fpath, skip=2, col_names = FALSE)
```

    ## New names:
    ## * `` -> ...1
    ## * `` -> ...2
    ## * `` -> ...3
    ## * `` -> ...4
    ## * `` -> ...5
    ## * ...

``` r
names(the_data) <- mynames
rm(mynames)
```

``` r
the_data <- the_data %>%
  select(-count, -time, -datetime)  %>%       # datetime and time contain the same data
  mutate(dt = make_datetime(year, month, day, 
                            hour, 0, tz = "America/New_York")) %>%
  rename(datetime = dt) %>%
  mutate(thedate  = as.Date(datetime),
         doy      = as.numeric(format(datetime, format = '%j')),
         tstamp   = paste0(year, '/', sprintf("%02d", month), '/',
                           sprintf("%02d", day), ' ', sprintf("%02d", hour)),
         Month = factor(month, labels = month.abb)) %>%
  arrange(datetime)              # Confirm that data are in chronological order
```

### CO2SYS Results

We ran CO2SYS in Python, principally to calculate estimated pH under the
total pH scale. Here we load it and use a left join by timestamp to add
the data to the principal data set.

``` r
niecefldnm <- 'PyCO2SYS_Calc'
niece  <- file.path(sibling, niecefldnm)

fn    <- 'focbco2sys_out.csv'
fpath <- file.path(niece,fn)


ph_tot_data <- read_csv(fpath, 
    col_types   = cols(month = col_integer(), 
        year    = col_integer(), 
        day     = col_integer(), 
        hour    = col_integer(), 
        temp    = col_number(), 
        sal     = col_number(), 
        pco2    = col_number(), 
        ph      = col_number(), 
        omega_a = col_number(), 
        omega_c = col_number(), 
        ta      = col_number(), 
        dic     = col_number(),
        ph_tot  = col_number())) %>%
  mutate(tstamp = paste0(year, '/', sprintf("%02d", month), '/',
                         sprintf("%02d", day), ' ', sprintf("%02d", hour))) %>%
  select(ph_tot, tstamp)
```

    ## Warning: One or more parsing issues, see `problems()` for details

### Merge pH (Total Scale) data into primary data

Note this assumes there are no duplicate time stamps…

``` r
the_data <- the_data %>%
  left_join(ph_tot_data, by='tstamp') %>%
  select(-tstamp)
rm(ph_tot_data)
```

### Add A Variable for the Season

``` r
the_data <- the_data %>%
  mutate(Season = recode_factor(month,
                                `1`  = 'Winter',
                                `2`  = 'Winter',
                                `3`  = 'Spring',
                                `4`  = 'Spring',
                                `5`  = 'Spring',
                                `6`  = 'Summer',
                                `7`  = 'Summer',
                                `8`  = 'Summer',
                                `9`  = 'Fall',
                                `10` = 'Fall',
                                `11` = 'Fall',
                                `12` = 'winter'))
```

## Takahashi et al. 2002 Relationships

Here we follow a formula for calculating a “Temperature Corrected”
pCO<sub>2</sub>, which is derived from methods in Takahashi et al. 2002.
The “temperature corrected” version adjusts for the thermodynamic effect
of temperature on pCO<sub>2</sub>.

> Takahashi, Taro & Sutherland, Stewart & Sweeney, Colm & Poisson, Alain
> & Metzl, Nicolas & Tilbrook, Bronte & Bates, Nicholas & Wanninkhof,
> Rik & Feely, Richard & Chris, Sabine & Olafsson, Jon & Nojiri,
> Yukihiro. (2002). Global sea-air CO2 flux based on climatological
> surface ocean pCO2, and seasonal biological and temperature effects.
> Deep Sea Research Part II: Topical Studies in Oceanography. 49.
> 1601-1622. 10.1016/S0967-0645(02)00003-6.

Takahashi et al. 2002 Used direct calculation of “temperature corrected”
pCO<sub>2</sub> as a surrogate for changes in CO<sub>2</sub>
concentration, and conversely, estimates of “expected” thermal
pCO<sub>2</sub>, as estimates of the magnitude of the fluctuations in
pCO<sub>2</sub> one would expect to see due to temperature alone, if
there were no changes in \[CO<sub>2</sub>\].

The Takahashi et al. 2002 equations are as follows:

#### “Expected pCO<sub>2</sub>” at Observed Temperature

(*p**C**O*<sub>2</sub> at *T*<sub>obs</sub>) = (*p**C**O*<sub>2</sub>)<sub>obs</sub> × *e**x**p*(0.0423(*T*<sub>obs</sub> − *T*<sub>mean</sub>)

#### “Temperature Corrected” pCO<sub>2</sub>

(*p**C**O*<sub>2</sub> at *T*<sub>mean</sub>) = (*p**C**O*<sub>2</sub>)<sub>obs</sub> × *e**x**p*(0.0423(*T*<sub>mean</sub> − *T*<sub>obs</sub>)

### Calculations

We calculate the “temperature corrected” time series as calculated in
Takahashi et al. “Temperature Corrected” pCO<sub>2</sub> value
(“co2\_corr”) provides a trace of changes in pCO<sub>2</sub> that “would
have happened” in the absence of temperature changes. These reflect
changes in the concentration of CO<sub>2</sub>, which reflect a
combination of biology and diffusion of CO<sub>2</sub> between ocean and
atmosphere and advection past the sensor by tides and currents. Here we
adjust pCO<sub>2</sub> to a “standard temperature” of 12 degrees C. This
is slightly warmer than the observed annual average temperature. We use
12 degrees C principally for consistency with analysis of the CBEP / UNH
data.

``` r
t_ref = 12
the_data <- the_data %>%
  mutate(pco2_corr =  pco2*exp(0.0423*(t_ref-temperature)))# %>%
  #select(c(16, 17, 11, 10, 9, 19, 12,  18, 1:5, 7,  6, 20, 8, 21, 13:15))  # reorder for convenience
rm(t_ref)
```

## Calculate Diurnal Deviations

We calculate hourly deviation from daily averages for the principal OA
measures. This allows us to look at diurnal patterns.

``` r
diurnal_data <- the_data %>%
  group_by(thedate) %>%
  
  # Calculate sample sizes for each day.
  mutate(pco2_n      = sum(! is.na(pco2)),
         pco2_corr_n = sum(! is.na(pco2_corr)),
         ph_n        = sum(! is.na(ph)),
         ph_tot_n    = sum(! is.na(ph_tot)),
         omega_n     = sum(! is.na(omega_a))) %>%
  
  # Calculate centered but not scaled values, day by day.
  mutate(pco2_res      = scale(pco2, scale = FALSE),
         pco2_corr_res = scale(pco2_corr, scale = FALSE),
         ph_res        = scale(ph, scale = FALSE),
         ph_tot_res    = scale(ph_tot, scale = FALSE),
         omega_res     = scale(omega_a, scale=FALSE)) %>%
  ungroup() %>%
  
  # Replace data from any days with less than 20 hours of data with NA
  mutate(pco2_res      = ifelse(pco2_n>=20, pco2_res, NA),
         pco2_corr_res = ifelse(pco2_corr_n>=20, pco2_corr_res, NA),
         ph_res        = ifelse(ph_n>=20, ph_res, NA),
         ph_tot_res    = ifelse(ph_tot_n>=20, ph_res, NA),
         omega_res     = ifelse(omega_n>=20, ph_res, NA)) %>%
  
  # Delete the daily sample size variables
  select(-contains("_n")) %>%
  mutate(Month = factor(month, levels = 1:12, labels=month.abb))
```

# Base Graphics

## Constants for Axis Labels

``` r
monthlengths <-  c(31,28,31, 30,31,30,31,31,30,31,30,31)
cutpoints    <- c(0, cumsum(monthlengths)[1:12])[1:12]
```

## Seasonal Profiles

These graphs combine data from multiple years to generate a picture of
seasonal conditions across multiple years. Since data coverage is
inconsistent year to year, data for some times of year are derived from
just one or two years, which could bias the results.

### Temperature Corrected pCO2

It’s not technically OK to show a reference line on a figure with
temperature-corrected pCO2. The equilibrium between \[co<sub>2</sub>\]
and fugacity is temperature dependent.

``` r
plt <- ggplot(the_data, aes(doy, pco2_corr)) +
  geom_point(aes(color = factor(year)), alpha = 0.1) +
  xlab('') +
  ylab(expression (Corrected~pCO[2]~(mu*Atm))) +
  scale_color_manual(values=year_colors, name='Year') +
  scale_x_continuous(breaks = cutpoints, labels = month.abb) +
  guides(colour = guide_legend(override.aes = list(alpha = 1))) +
  theme_cbep() +
  theme(axis.text.x=element_text(angle=90, vjust = 1.5))
plt
```

    ## Warning: Removed 10799 rows containing missing values (geom_point).

![](FOCB_OA_Revised_Graphics_files/figure-gfm/pc02_by_doy-1.png)<!-- -->

``` r
ggsave('figures/pco2Seasonal_focb.pdf', device=cairo_pdf, width = 7, height = 5)
```

    ## Warning: Removed 10799 rows containing missing values (geom_point).

This shows much less obvious seasonality than the CBEP / UNH site.

### pH (Total Scale)

``` r
plt <- ggplot(the_data, aes(doy, ph_tot)) + 
  geom_point(aes(color = factor(year)),alpha = 0.1) +
  xlab('') +
  ylab('pH') +
  scale_color_manual(values=year_colors, name='Year') +
  scale_x_continuous(breaks = cutpoints, labels = month.abb) +

  scale_y_continuous(limits = c(7.5, 8.5), breaks = c(7.5, 7.75, 8.0, 8.25, 8.5)) +
  
  guides(colour = guide_legend(override.aes = list(alpha = 1))) +
  theme_cbep() +
  theme(axis.text.x=element_text(angle=90, vjust = 1.5))
plt
```

    ## Warning: Removed 909 rows containing missing values (geom_point).

![](FOCB_OA_Revised_Graphics_files/figure-gfm/ph_by_doy-1.png)<!-- -->

``` r
ggsave('figures/phSeasonal_focb.pdf', device=cairo_pdf, width = 7, height = 5)
```

    ## Warning: Removed 909 rows containing missing values (geom_point).

### Aragonite Saturation State

``` r
plt <- ggplot(the_data, aes(doy, omega_a)) + 
  geom_point(aes(color = factor(year)), alpha = 0.1) +
  geom_hline(aes(yintercept = 1), lty = 'solid', color = 'gray') +
  geom_text(aes(x=0, y=1.1, label= 'Omega = 1.0', hjust = 0), size=3) +
  
  xlab('') +
  ylab(expression(Omega[a])) +
  
  scale_color_manual(values=year_colors, name='Year') +
  scale_x_continuous(breaks = cutpoints, labels = month.abb) +
  
  guides(colour = guide_legend(override.aes = list(alpha = 1))) +
  
  theme_cbep() +
  theme(axis.text.x=element_text(angle=90, vjust = 1.5))
  
plt
```

    ## Warning: Removed 10800 rows containing missing values (geom_point).

![](FOCB_OA_Revised_Graphics_files/figure-gfm/omega_by_doy-1.png)<!-- -->

``` r
ggsave('figures/omegaSeasonal_focb.pdf', device=cairo_pdf, width = 7, height = 5)
```

    ## Warning: Removed 10800 rows containing missing values (geom_point).

# Daily Patterns

This takes so long, we wrapped it in an IF to avoid rerunning it if we
either have a loaded version or a saved version. Note that this means if
you change data or model specifications, it will not update your results
unless you delete the saved versions manually.

## pCO<sub>2</sub>

### Model

This model takes about 10 minutes to run.

``` r
if (! exists('pco2_diurnal_gam')) {
  if (file.exists('pco2_diurnal_gam.rds')) {
    pco2_diurnal_gam <- readRDS('pco2_diurnal_gam.rds')
    }
  else {
    print(system.time(pco2_diurnal_gam <- gamm(pco2_corr_res ~ s(hour, 
                                                                 by = Season,
                                                                 bs = 'cc', 
                                                                 k=6), 
                                     correlation = corAR1(form = ~ 1 | Season),
                                     data = diurnal_data)))
    saveRDS(pco2_diurnal_gam, 'pco2_diurnal_gam.rds') 
    }
}
```

### Predictions

``` r
newdat <- expand.grid(hour = 0:23, 
                      Season = c('Winter', 'Spring', 'Summer', 'Fall'))
p <- predict(pco2_diurnal_gam$gam, newdata=newdat, type = 'link', se = TRUE)
newdat <- newdat %>%
  mutate(pred = p$fit, se = p$se.fit)
```

### Corrected pCO2 Graphic

``` r
plt <- newdat %>%
  ggplot(aes(x = hour)) +
  geom_ribbon(aes(ymin = pred - 1.96*se, 
                  ymax = pred +1.96*se, 
                  fill = Season), 
              alpha = 0.5) +
  theme_cbep(base_size = 12) +
  xlab('Hour of Day') +
  ylab(expression (atop(Corrected~pCO[2]~(mu*Atm), Difference~From~Daily~Average))) +
  scale_fill_manual(values = seasonal_colors,  name = "") + 
  theme(legend.key.width = unit(0.25,"in"),
        legend.text      = element_text(size = 10)
        ) +
  scale_x_continuous(breaks = c(0,6,12,18,24)) #+
plt
```

![](FOCB_OA_Revised_Graphics_files/figure-gfm/pco2_figure-1.png)<!-- -->

``` r
ggsave('figures/dailyCO2bymonth_focb.pdf', 
       device=cairo_pdf, width = 4, height = 4)
```

## pH

### Model

This model takes over 30 minutes to run.

``` r
if (! exists('ph_diurnal_gam')) {
  if (file.exists('ph_diurnal_gam.rds')) {
    ph_diurnal_gam <- readRDS('ph_diurnal_gam.rds')
    }
  else {
    system.time(ph_diurnal_gam <- gamm(ph_res ~ s(hour, 
                                                  by = Season, 
                                                  bs = 'cc', 
                                                  k = 6), 
                                     correlation = corAR1(form = ~ 1 | Season), 
                                     data = diurnal_data))
    saveRDS(ph_diurnal_gam, 'ph_diurnal_gam.rds') 
    }
}
```

### Predictions

``` r
newdat <- expand.grid(hour = 0:23, 
                      Season = c('Winter', 'Spring', 'Summer', 'Fall'))
p <- predict(ph_diurnal_gam$gam, newdata=newdat, type = 'link', se = TRUE)
newdat <- newdat %>%
  mutate(pred = p$fit, se = p$se.fit)
```

### pH Graph

``` r
plt <- newdat %>%
  ggplot(aes(x =hour)) +
  geom_ribbon(aes(ymin = pred - 1.96 * se, 
                   ymax = pred + 1.96 * se,
                   fill=Season), alpha = 0.5) +
  theme_cbep(base_size = 12) +
  theme(legend.key.width = unit(0.25,"in"),
        legend.text      = element_text(size = 10)
        ) +
  xlab('Hour of Day') +
  ylab(expression (atop(pH, Difference~From~Daily~Average))) +
  scale_fill_manual(values = seasonal_colors, name = "") + 
  
  theme(legend.key.width=unit(0.25,"in")) +
  scale_x_continuous(breaks = c(0,6,12,18,24)) #+
plt
```

![](FOCB_OA_Revised_Graphics_files/figure-gfm/phfigure-1.png)<!-- -->

``` r
ggsave('figures/dailyphbymonth_focb.pdf', 
       device=cairo_pdf, width = 4, height = 4)
```
