
# Cleaning step

The "cleaning" step consists in deleting objects located outside of the country's extent and more than a certain distance away from its international boundaries. The default distance used is 5 meters.

This means that in OME2, we consider that objects provided by a country and located less than 5 meters away from the international boundary (even if they are in the neighbouring country's extent) can be kept. Objects located in the neighbouring country and more than 5 meters away from the international boundary are deleted by the cleaning step.

The cleaning step needs to be applied once new data has been integrated in the OME2 database and once the OME2 version of its international boundaries have been defined.

The cleaning step works like most steps in the OME2 process. At first, the data to be processed is stored in the "official" table (e.g. tn.road_link). The cleaning step is divided into 3 sub-steps:
* Sub-step 1: the relevant data is extracted in the corresponding working table (e.g. road_link_w). To do that, an extract distance needs to be defined (see below).
* Sub-step  2: the cleaning tool is applied to the working table (e.g. road_link_w) -> the objects located in the wrong country and more than 5 meters away from the boundary are deleted.
* Sub-step  3: the result is integrated in the original table (e.g. tn.road_link).

A single command line enables to perform the three sub-steps.

## Define the parameters
The cleaning tool needs two parameters: the extract distance and the cleaning distance.

#### Extract distance
The extract distance corresponds of the size of the buffer which will be created around the international boundary to extract the data for cleaning. It needs to be as small as possible in order to extract as little data as possible (so that the tool can run more efficiently) but large enough to contain all the objects located in the wrong country which need to be deleted.

To determine the extract distance, you can open the data in QGIS and approximately measure the distance between the international boundaries and the farthest objects. The extract distance parameter should be slightly higher than the distance measured in QGIS.

![Extract_distance_QGIS](https://github.com/openmapsforeurope2/OME2/blob/main/docs/images/Extract_distance_QGIS.png)

In the example above, the distance measured between the boundary and the farthest objects is ~17308 meters. To be safe, an extract distance of 20000 meters can be used.

Once the extracting distance has been decided, it needs to be indicated in the [configuration file](https://github.com/openmapsforeurope2/data-tools/blob/main/config/conf.json) used by the cleaning tool:
- if it is the same as the default value, it does not need to be added.
- if it is different from the default value, it needs to be added in the "extraction_distance" section corresponding to the theme to be processed (hy or tn).
<img width="567" height="480" alt="image" src="https://github.com/user-attachments/assets/3dcdc887-2228-4b13-bc26-ac019a3bb45e" />


#### Cleaning distance
The default value for the cleaning distance is 5 meters. This value was chosen empirically while testing the edge-matching process. 
There has been no reason to use a different value so far.

It is however possible to modify the cleaning distance in the [configuration file](https://github.com/openmapsforeurope2/data-tools/blob/main/config/conf.json):
<img width="505" height="480" alt="image" src="https://github.com/user-attachments/assets/935d1705-e3c0-47f4-a215-980de8e06200" />

Please note that the new cleaning distance will be applied to all themes and countries.

## Run the command line

Ex 1: clean French road_link on the boundaries with be and lu
~~~
python3 script/clean.py -c path/to/conf.json -b lu -b be -T tn -t road_link fr
~~~

Ex 2: clean the French tn theme (all tables) on all of France's international boundaries
~~~
python3 script/clean.py -c path/to/conf.json -a -T tn fr
~~~

The full documentation for the clean tool is accessible [here](https://github.com/openmapsforeurope2/data-tools/tree/main).

/!\ OBSOLETE
The command lines and parameters used during the production of the HVLSP 2.0 and 3.0 can be found in the [clean_hy_tn.sh](https://github.com/openmapsforeurope2/data-tools/blob/main/use_case/clean_hy_tn.sh) file in the data-tools/use_case repository.

