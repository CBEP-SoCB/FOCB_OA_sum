# Carbonate Chemistry Data From PyCO2SYS
Friends of Casco Bay collects pH data using the pH sensors on Exo sondes from
YSI.   These are "conventional" ion-selective pH electrodes, and thus measure pH
on the "National Bureau of Standards" or NBS pH scale.  However, in oceanography,
pH is often measured on other pH scales. In particular, UNH used a different 
type of sensor to collect pH data at the ocean acidification monitoring location 
they managed on behalf of CBEP. At that site, pH was recorded on the "Total"
pH scale. The two scales are not fully compatible, and can differ by more than a
tenth of a pH point.

We want to be able to compare pH observations at the FOCB site and the UNH site, 
so we need to figure out how to convert from the NBS scale to the Total Scale.
The conversion from one to the other is not simple in estuarine waters.  The
challenge is that the conversion depends on the activity of other cations in the 
water, which can can vary significantly in estuaries. After discussions with 
Chris Hunt, of UNH, and others, we determined that the best way to make the 
conversion would be to calculate the full carbonate chemistry implied by the 
FOCB data and then use the derived carbonate equilibrium to estimate the 
(unmeasured) pH on the Total Scale.

This is an emerging area of scientific practice, and if we were to redo these
analyses today, we might make different choices.  the pH - pCO~2~ pair on which
the FOCB carbonate equilibria were calculated does not perform all that well.
Estimating pH this way may have introduced more error into our calculations than
would have been added using a (technically incorrect, but simple) linear
correction.  In future, we expect data from FOCB sites to include estimates of
alkalinity based on salinity measurements, which should lead to more robust
estimation of carbonate equilibria.

## Why Python?
The standard tool for calculating carbonate equilibria is `CO2SYS`, developed
principally in MATLAB.  MATLAB is in widespread use in the oceanography 
community, but it is less commonly used by inshore ecologists.  `CO2SYS` has
been successfully ported to a number of other environments, including Excel, R,
and Python.  Each version exists in a quasi-independent state, with its own
irregular update process.  Most updated appear first in MATLAB, and gradually
filter out into other versions over time.  Not all versions have the same
capabilities.

The primary carbonate calculator in R, `seacarb`, is based on `CO2SYS`, but it
cannot handle pH measured on the NBS scale, which is what FOCB uses in their
monitoring.  Thus we can not use it to calculate the FOCB carbonate equilibria,
and thus can not use it to convert the pH measurements to a Total scale.

This port of `CO2SYS` to Python (PyCO2Sys) does allow use of the NBS pH scale.
So we fall back on somewhat rusty Python programming skills to calculate
estimated pH on the Total pH scale, for comparison to data from UNH.

## `focbco2sys_our.csv`
This is the data file output by our python code.  It includes raw hourly data derived
from FOCB's sonde and co-located pCO~2~ sensor, and also calculated carbonate 
parameters derived from that data including Omega aragonite, Omega calcite, 
total alkalinity, dissolved inorganic carbon, and our estimate of pH on the 
Total scale.

month        | Month of observation       | Integer 1 to 12, 1 = January
year         | Year of observation        | four digit integer
day          | Day of observation         | day of month integer 1-31
hour         | Hour of observation        | integer, 24 hour clock  
temp         | Water temperature          | Celsius
sal          | Salinity, based on conductivity  | PSU (roughly, PPT)
pco2         | Partial pressure of CO2    | uAtm
ph           | pH from sonde pH sensor    | NBS pH scale
omega_a      | Omega aragonite            | Dimensionless
omega_c      | Omega calcite              | Dimensionless
ta           | Total Alkalinity (calculated) |  uM/kg	
dic          | Dissolved inorganic carbon (calculated)| uM/kg
ph_tot       | pH (calculated)            | Total pH scale

