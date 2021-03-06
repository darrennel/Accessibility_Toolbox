# Accessibility Toolbox for R and ArcGIS

## Overview
This repository contains the tools, code, and data to support the paper *Accessibility Toolbox for R and ArcGIS* published in *Transport Findings* at DOI [10.32866/8416](https://doi.org/10.32866/8416).

<img width="900" alt="fig_2" src="https://github.com/higgicd/Accessibility_Toolbox/blob/master/assets/fig_2.jpg">

The Accessibility Toolbox contains two tools. The first is the **Accessibility Calculator** Python toolbox for ArcGIS Pro and 10x that simplifies the steps involved in a place-based accessibility workflow. Two versions of the tool are included in the toolbox: the first outputs a single origin-destination matrix while the second segments origins into smaller batches and overwrites the origin-destination matrix across iterations to save memory and disk space for large analyses.

The second is an interactive **Accessibility Toolbox** R Notebook that visualizes the 5 impedance functions from Kwan (1998) and 28 impedance measures to aid in the selection and customization of accessibility cost functions. Customized parameters can be implemented in the ArcGIS tool's python code.

## Version History
- ```v2.1``` added the *OD Cost Matrix Calculator for ArcGIS Pro 2.4 Multiprocessing* for the rapid calculation of raw origin-destination cost matrices
  - takes full advantage of your multi-core PC
  - joins back in the original origins ID for further analysis
  - note - the final merge of all the worker matrices might take a while for very large analyses with billions of rows as this operation is still single-threaded
- ```v2.0``` the *Accessibility Calculator* for ArcGIS Pro has been entirely rewritten and is now **4 to 5 times faster**
  - introduced a new version of the tool called *Accessibility Calculator Pro MP* for ArcGIS Pro 2.4 or above that takes advantage of **parallel processing** and data access cursors for dramatic improvements in processing time
  - main functions are now moved to the ```access_calc_main.py``` file which can be run from either the Python toolbox or command line
  - impedance parameters are now stored in their own separate ```parameters.py``` file and streamlined for easy editing
  - the tool now includes ```.xml``` files for help pop-ups in the geoprocessing window
  - the *Accessibility Calculator* for ArcGIS 10x and earlier versions of ArcGIS Pro have been moved to the ```legacy_tools``` folder
- ```v1.1``` altered the *Accessibility Calculator* for 10x and Pro:
  - introduced a new version of the *Accessibility Calculator for ArcGIS Pro 2.4* that uses the new [arcpy.nax](https://pro.arcgis.com/en/pro-app/arcpy/network-analyst/what-is-the-network-analyst-module.htm) Network Analyst module introduced in ArcGIS Pro 2.4. This module stores intermediate network analyst data ```in_memory``` and promises to be [significantly faster](https://pro.arcgis.com/en/pro-app/arcpy/network-analyst/choosing-between-the-two-modules-arcpy-nax-versus-arcpy-na-.htm). To take advantage of this, the new 2.4 version of the *Accessibility Calculator* also stores its intermediate data ```in_memory``` and together these changes result in about a 25% improvement in speed versus the old version of the tool (this on a PC with an SSD - the savings will be even more dramatic with a spinning drive). However, if you are memory constrained, stick to the older version of the tool.
  - fixed an error related to assigning sequential IDs in the *batch calculator* for Pro introduced after updating to ArcGIS Pro 2.4
  - changed the ```search criteria``` and ```search query``` parameters to valueTables to make it much easier to specify search restrictions
  - changed the ```search tolerance``` parameters to proper linear unit data types
  - changed the ```departure time``` parameter to a proper datetime format for the Pro toolboxes
- ```v1.0``` original release alongside the paper in *Transport Findings* 

## Calculating Accessibility in ArcGIS
The Accessibility Calculator has several inputs:  
<img align="right" width="275" height="1250" src="https://github.com/higgicd/Accessibility_Toolbox/blob/master/assets/toolbox.png">

*Network Details*
- **Input Network Dataset**: the input Network Analyst network dataset for your analysis
- **Travel Mode** (Pro): the travel mode for your analysis from your network dataset
- **Cutoff Value**: the travel time value at which to stop searching for destinations for a given origin
- *Departure Time* (optional): a time of departure from the origins; useful for analyses with traffic or GTFS transit scheduling
  - see the ArcGIS [Make OD Cost Matrix Analysis Layer](https://pro.arcgis.com/en/pro-app/tool-reference/network-analyst/make-od-cost-matrix-analysis-layer.htm) reference for more guidance and examples on setting a *departure time* parameter
- **Impedance Measure**: a selection of one or more impedance measures; refer to the *R Notebook* for guidance on selecting a measure

*Origins*
- **Origins**: a feature class representing origin locations *i*; can be point or polygon
- **Origins ID Field**: a unique identifier for your input origins; can be any type of field
- **Origins Network Search Tolerance**: the search tolerance for locating the input features on the network; features that are outside the search tolerance are left unlocated
- **Origins Network Search Criteria**: restricts the search to particular source feature classes in your network dataset; useful if you don't want to find features that may be unsuited for a network location
  - for example, if you have created a transit network using the [Add GTFS to Network Dataset](https://esri.github.io/public-transit-tools/AddGTFStoaNetworkDataset.html) tool and want your origins to locate on streets while avoiding other feature classes like transit lines, you could input these expressions (using default names for the GTFS tool) into the value table: ```Streets_UseThisOne``` with a snap type of ```SHAPE```; ```Stops``` and ```NONE```; ```Stops_Snapped2Streets``` and ```NONE```; ```Transit_Network_ND_Junctions``` and ```NONE```; ```Connectors_Stops2Streets``` and ```NONE```; and ```TransitLines``` and ```NONE``` (see example for [Pro](https://github.com/higgicd/Accessibility_Toolbox/blob/master/assets/search_criteria_query_pro.png))
  - see the ArcGIS [Add Locations](https://pro.arcgis.com/en/pro-app/tool-reference/network-analyst/add-locations.htm) reference for more guidance and examples on setting a *search_criteria* parameter
- *Origins Network Search Query* (optional): specifies a query to restrict the search to a subset of the features within a network source feature class; useful if you don't want to locate on particular features
  - for example, if you do not want your input origins to locate on major highways, you could input ```Streets``` with the expression ```FREEWAY=0```
  - in the example data, if you do not want origins to locate on tunnels or bridges, you could input the expressions ```NYC_OSM_Walk``` and ```tunnel<>'yes'``` and ```NYC_OSM_Walk``` and ```bridge<>'yes'``` into the value table (see example for [Pro](https://github.com/higgicd/Accessibility_Toolbox/blob/master/assets/search_criteria_query_pro.png))
  - see the ArcGIS [Add Locations](https://pro.arcgis.com/en/pro-app/tool-reference/network-analyst/add-locations.htm) reference for more guidance and examples on setting a *search_query* parameter

*Destinations*
- **Destinations**: a feature class representing destination locations *j*; can be point or polygon
- **Destinations ID Field**: a unique identifier for your input destinations; can be any type of field
- **Destination Opportunities Field**: a numeric field containing the opportunities *Oj* available at the destination, such as the number of jobs
- **Destinations Network Search Tolerance**: same notes as the *Origins Network Search Tolerance*
- **Destinations Network Search Criteria**: same notes as the *Origins Network Search Criteria*
- *Destinations Network Search Query* (optional): same notes as the *Origins Network Search Query*

*General Settings*
- **Output Work Folder**: the folder where the output geodatabase and multiprocessing workers folder will be created; working files generated during large analyses can require many gigabytes of disk space but they are automatically deleted once the tool finishes
- **Name of Output Analysis Geodatabase**: name for the output geodatabase containing the scratch working files and the final tool output
- **Origins Maximum Batch Size**: maximum size of each batch of origins for multiprocessing and controlling memory/disk use; some optimization may occur if the number of observations can be spread over the available processes within your set maximum batch size (```(n/(cpus-1))+1 if n/(cpus-1) <= batch size```), otherwise it adheres to ```n/batch size```
- *Delete OD lines where i = j?* (optional): if selected, the tool will delete any origin-destination lines or pairs where the origin was the same as the destination; useful if you only want to calculate access to opportunities that are external to the origins
- *Join output back to origins?* (optional): if selected, joins the accessibility output back to the input origins

## Selecting an Impedance Function in R
Using the interactive R Notebook, users can explore 5 impedance functions: 
- inverse power
- negative exponential
- modified Gaussian
- cumulative opportunities rectangular
- cumulative opportunities linear

Each function is specified with several different impedance parameters for a total of 28 different impedance measures.

## Customizing or Adding Your Impedance Measure to the Tool
Users can add or change the impedance functions in the ArcGIS Accessibility Calculator by editing the ```parameters.py``` file. This file specifies the different impedance functions and stores relevant parameters in the dictionary *p*. Using the examples of the ```POW0_8``` and ```CUMR10``` impedance measures, each dictionary entry contains the following elements:

```
p = {
    "POW0_8": {"f": "power", "b0": 0.8}, 
    "CUMR10": {"f": "cumr", "t_bar": 10}
    }
```

- ```POW0_8``` and ```CUMR10```: the names of the impedance measures
- ```"f"```: the key ```f``` indexes the value of the impedance function for the impedance measure; in the case of ```POW0_8```, the ```"pow"``` string refers to the inverse power function; for ```CUMR10``` the function is ```"cumr"```. Available functions are:
  - ```power```: inverse power
  - ```neg_exp```: negative exponential
  - ```mgaus```: modified Gaussian
  - ```cumr```: cumulative opportunities rectangular
  - ```cuml```: cumulative opportunities linear
- ```"b0"```: the key ```b0``` indexes the value of the beta parameter for the impedance measure that will be input into the impedance function
  - in the case of the ```POW0_8``` measure, ```0.8``` refers to a beta value of 0.8
- ```"t_bar"```: the key ```"t_bar"``` indexes the value of the travel time window for the cumulative impedance functions
  - in the case of the ```CUMR10``` measure, ```10``` refers to a travel time window of 10 minutes

- if you make any additions of functions to the ```parameters.py``` file, you **must** update the list of impedance functions in the ```Accessibility_Toolbox_Pro_MP.pyt``` Python Toolbox. The list of function names starts on Line 223.

## References

Higgins, C. D. (2019). Accessibility toolbox for R and ArcGIS. *Transport Findings*. https://doi.org/10.32866/8416

Kwan, M. P. (1998). Space-time and integral measures of individual accessibility: A comparative analysis using a point-based framework. *Geographical Analysis*, 30(3), 191-216. https://doi.org/10.1111/j.1538-4632.1998.tb00396.x

## License
<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>.
