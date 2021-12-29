# Friends of Casco Bay Coastal Acidification Data

## "Continuos" Monitoring Data
"CMS1 Data through 2019.xlsx" contains data logger data collected by FOCB.
"CMS1" refers to "Continuous MOnitoring Station 1, located at the Chebeague 
Island Ferry dock, on the ocean side of Cousins Island.  The data were collected 
with a YSI Sonde and a separate C-Sense pCO~2~ sensor.  The same data is also
used in the
[FOCB_WQ_sum repository](https://github.com/CBEP-SoCB/FOCB_WQ_sum.git).

Column Name     | Contents                               | Units                         
----------------|----------------------------------------|------
Date	          | Dates, in a mixture of Excel Dates, Times, and Character Strings |
Time	          | Time of Day, similarly, in a mixture of formats  |
Water Depth     | Water depth (from pressure transducer on sonde)  | m
Temperature     | Temperature                          |   	C
Salinity	      | Salinity based on conductivity, roughly in PPT | PSU
DO	            | Dissolved oxygen                     | mg/l
DO%             | Percent oxygen saturation            |	%
pH	            | pH, measured with a pH electrode, this on NBS scale | Unitless
Chl             | Chlorophyll-A (from florescence)     |	ug/l
pCO2            | Partial pressure of CO~2~            |	ppm
Month	          | Month                                | as integer 1 through 12   
Year	          | Year,                                | Four digit integer
Day             | Day of the month                     |	Integer
Hour	          | Time of day, to nearest hour         | 0 to 23
TA	            | Total alkalinity  (Calculated from pH and pCO~2~)  | uM/kg
DIC             | Dissolved Inorganic carbon (Calculated)    |	uM/kg
Omega Aragonite |	Solubility quotient aragonite (Calculated  | Unitless
