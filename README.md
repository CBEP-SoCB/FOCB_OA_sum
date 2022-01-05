# Friends of Casco Bay Ocean Acidification Data
<img
    src="https://www.cascobayestuary.org/wp-content/uploads/2014/04/logo_sm.jpg"
    style="position:absolute;top:10px;right:50px;" />
    
Friends of Casco Bay has been monitoring water quality in Casco Bay for nearly
thirty years. in 2016, they began collecting continuous data from a monitoring
location on Cousins Island, including data on important acidification
parameters.  Here we focus on coastal acidification, but FOCB data is also
highlighted in related
[water quality repositories](https://github.com/CBEP-SoCB/FOCB_WQ_sum.git).

# Statement of Purpose
CBEP is committed to the ideal of open science.  Our State of the Bay data
archives ensure the science underlying the 2020 State of the Bay report is
documented and reproducible by others. The purpose of these archives is to
release raw data and data analysis code whenever possible to allow others to
review, critique, learn from, and build upon CBEP science.

# Archive Structure
 CBEP 2020 State of the Bay data analysis repositories are divided into from two
 to four sub-folders.  All archives contain an "Original_Data" folder and at
 least one other folder, usually "Graphics" or "Analysis". Other folders, often
 a "Derived_Data" folder are included if necessary.
 
 The archive contains the following sub-folders:

- Data.  Working data, with a "DATA_SOURCES.md"  and DATA_NOTES.md" files
  that document data sources and contents.
  
  - PyCO2SYS_Calc. This subfolder contains a python script to convert pH
    observations from the NBS scale to the Total Scale, for comparison with data
    collected by UNH scientists. Results as produced via the Python CO2SYS
    calculations are provided as a CSV file.

- Analysis_And_ Graphics.  Contains R Notebooks proceeding through 
  the data analysis and development of related graphics.  Raw copies of
  graphics in \*.pdf format are contained in a "figures" sub-folder. These
  graphics may differ from graphics as they appear in final State of Casco Bay
  graphical layouts.
  

# Summary of Data Sources
## Primary OA Data 
Friends of Casco Bay has been monitoring water quality in Casco Bay for nearly
thirty years. in 2016, they began collecting continuous data from a monitoring
location on Cousins Island, including data on important acidification
parameters.

Their "Cage of Science" monitoring apparatus consists of a YSI data sonde
(currently an "EXO" model), collecting data on primary water quality
parameters (including pH), and C-Sense pCO2 sensor, measuring pCO2.  Both are
housed in a modified lobster trap, which facilitates deployment and retrieval.

FOCB staff visit the site regularly, and swap out sensors and equipment as
needed to ensure a  near-continuous year-round data record.

Additional information on their continuous water quality monitoring program is
available on the FOCB web site.

The YSI sondes rely on glass electrode technologies to measure pH via measuring
electrical potential across an ion-selective pH electrode. It is important to
recognize that pH measured electrochemically is reported on a different scale
(the "NBS" scale) from pH measured using other technologies (e.g., 
colorimetrically, or using CFET technologies).

The values are not directly commensurable, and need to be converted before
comparisons.  Details are provided in the materials elsewhere in this archive.
In brief, we used a python script running PyCO2SYS to calculate revised
estimates of pH under the "total" pH scale.

(Because a number of assumptions are necessary to run the CO2SYS calculations,
we only used this to estimate pH, and used FOCB's estimates of all other
carbonate chemistry parameters, such as "Omega aragonite".)

## Supporting data
CBEP staff accessed data on weather and tides from NOAA, using published APIs
and simple python scripts. See the  
[CBEP_OA archive](https://github.com/CBEP-SoCB-Details/CBEP_OA) for details.

Data access scripts are also provided in the data analysis archives for
[Climate Change](https://github.com/CBEP-SoCB-Details/CDO-Portland-Jetport) and 
[Sea Level Rise](https://github.com/CBEP-SoCB-Details/Portland-SLR).
