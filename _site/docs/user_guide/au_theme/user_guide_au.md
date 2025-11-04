# Administrative units theme (AU)

## Summary
National administrative units have been provided by a country to the OME2 project. If neighbouring countries have also provided data, the OME2 project may have calculated technical international boundaries to be used by the project, based on the data provided by neighbouring countries, which are slightly different from the versions provided by each country. As a result, national administrative units are no longer consistent with the technical boundaries used in OME2.
Example (OME2 international boundary in light green) :

<img width="470" height="334" alt="image" src="https://github.com/user-attachments/assets/564f34b4-11ce-4e49-9628-44b34464d90f" />

The objective is to align national administrative units at all levels with the international boundaries agreed upon for OME2.

This will be done in two steps:
- Step 1: administrative units at the lowest level are aligned with the international boundaries, using the [au_matching](https://github.com/openmapsforeurope2/au_matching) tool.
- Step 2: administrative units from upper levels are recreated by merging lower level units (correctly aligned thanks to step 1), using the [au_merging](https://github.com/openmapsforeurope2/au_merging) tool. This step has to be applied on all upper levels successively.
- Step 3: fill the administrative_hierarchy table with an FME workbench [AU_manage_administrative_hierarchy.fmw](https://github.com/openmapsforeurope2/fme_workbenches/tree/main/AU).

At the end of these 3 steps, the production of the AU theme for the new country is finished.

## Pre-requisites
- The country's national data has been converted to the OME2 data model and integrated in the central database.
- Technical international boundaries have been set up with the neigbouring countries.

## Step 1: edge-match administrative units at the lowest level
This step needs to be performed on each country's lowest administrative level. The table below indicates, for each country, the lowest administrative level provided. The distance column indicates which distance is used to extract administrative units along boundaries (see below).

| country_code | country                        | distance | lowest_level | 
|--------------|--------------------------------|----------|--------------|
| ad           | Andorra                        | 1000     | 1            |
| at           | Austria                        | 1000     | 5            |
| be           | Belgium                        | 1000     | 5            |
| ch           | Switzerland                    | 1000     | 4            |
| cz           | Czech Republic                 | 1000     | 4            |
| dk           | Denmark                        | 1000     | 3            |
| es           | Spain                          | 1000     | 4            |
| fi           | Finland                        | 1000     | 5            |
| fr           | France                         | 1000     | 6            |
| li           | Liechtenstein                  | 1000     | 2            |
| lu           | Luxembourg                     | 1000     | 3            |
| nl           | The Netherlands                | 1000     | 3            |

#### 1.1. Extract objects around a country's boundaries for matching
The process is launched only on administratives units along international boundaries. It can be launched on all the country's international boundaries at once, or on a single boundary.
To extract the relevant administrative units, the border_extract tool from the data-tools repository is used. The data is automatically extracted in the *administrative_unit_area_<level>_w* table in the database's public schema.

<img width="356" height="276" alt="image" src="https://github.com/user-attachments/assets/b8db5495-f400-42a7-8b7f-9da5ea8134c3" />

Generic command line to extract administrative units along all the country <country_code>'s international boundaries:
```
python3 script/border_extract.py -c config/conf.json -T au -t administrative_unit_area_<level> -d <distance> <country_code> '#'
```
Example of an extraction for country <country_code> along its international boundary with <country_code_neighbour>:
```
python3 script/border_extraction.py -c config/conf.json -T au -t administrative_unit_area_<level> -b <country_code_neighbour> -d <distance> <country_code>
```

<u>Belgium:</u>
Example of an extraction around boundaries with several neighbouring countries: the first line extracts units along the be#nl boundary. In the next lines, the "-n" option indicates that the selection has to be added to the existing selection.
```
python3 script/border_extraction.py -c config/conf.json -T au -t administrative_unit_area_5 -b nl -d 1000 be
python3 script/border_extraction.py -c config/conf.json -T au -t administrative_unit_area_5 -b de -d 1000 -n be
python3 script/border_extraction.py -c config/conf.json -T au -t administrative_unit_area_5 -b lu -d 1000 -n be
```

#### 1.2. Matching
Once the relevant data has been extracted in the administrative_unit_area_<lowest_level>_w table, the matching process itself can be launched with the au_matching tool. This process with align administrative units contained in the work table with the OME2 international boundary.
```
bin/au_matching --c data/config/epg_parameters.ini --t <working_schema>.administrative_unit_area_<lowest_level>_w --cc <country_code>
```
#### 1.3. Interactive check
Check in the log folder whether invalid polygons have been generated by the matching process. If there are some, these objects are recorded in a shapefile named *resulting_polygons_not_valid.shp*. In QGIS, find the corresponding polygons in administrative_unit_area_<lowest_level>_w and correct them manually.

#### 1.4. Integrate modifications in the main table
After step 1.3, the administrative units which had been extracted in the administrative_unit_area_<lowest_level>_w table should be aligned with the international boundaries and valid. They can now be reintgrated in the official administrative_unit_area_<lowest_level> table with the "integrate" tool from the data-tools repository.
```
python3 script/integrate.py -c config/conf.json -T au -t administrative_unit_area_<lowest_level>
```

## Step 2: edge-match administrative units at all other levels
Upper level administrative units are recreated by merging AUs at a lower level, using the [au_merging](https://github.com/openmapsforeurope2/au_merging) tool on each level successively. 

Please note that the last complete level should be used as reference for merging. For instance, in France, level 5 does not cover the whole territory. Therefore, au.administrative_unit_area_4 will be recreated using au.administrative_unit_area_6 as reference:
<pre>au_merging --c path/to/config/epg_parameters.ini --s au_administrative_unit_area_6 --t administrative_unit_area_4_w --cc fr</pre>

The following steps need to be repeated for all upper administrative levels, going from lower to upper. For instance, if the lowest level is 4, these steps have to be performed three times successively on levels 3, 2 and 1. In the following command lines, the level to be processed is <n>. The level to be used as reference is named <ref>: as mentioned above, the <ref> level might be the one directly below level <n> if it is complete, or a lower level.

Please note that, since the polygons representing administrative units get bigger at each level, processing times increase as well. The process for level 1 for large countries such as France or Spain might take several hours.

#### 2.1. Extract objects around a country's boundaries
In the same way as step 1.1, administrative units at level n are extracted along international boundaries. They are stored in the administrative_unit_area_<n>_w working table, in the working schema.
```
python3 script/border_extract.py -c conf.json -T au -t administrative_unit_area_<n> -d <distance> <country_code> '#'
```
Example:

<img width="375" height="290" alt="image" src="https://github.com/user-attachments/assets/5f796d67-0fef-47d2-af1d-c64945cbe3f9" />


#### 2.2. Merging
Once the relevant data has been extracted in the administrative_unit_area_<n>_w table, the merging process itself can be launched. For each administrative unit from level <n>, the process detects and merge the administrative units from level <ref> which are part of it. Since these administrative units are already aligned with the boundaries, the resulting geometries for level n become aligned as well.  
```
bin/au_merging --c data/config/epg_parameters.ini --s <administrative_schema>.administrative_unit_area_<ref> --t <working_schema>.administrative_unit_area_<n>_w --cc <country_code>
```
#### 2.3. Interactive check
Invalid or small polygons may be generated by the merging process and recorded in log/small_or_slim_surface.shp. They need to be corrected in QGIS before running sub-step 2.4.
Please note that small_or_slim_surface.shp often contains a lot of small islands. Only small polygons generated inside or on the edges of merged polygons need to be corrected. They can be detected visually in QGIS using an adapted styling.

<img width="949" height="660" alt="image" src="https://github.com/user-attachments/assets/aeabf0b7-2ad4-406b-8193-131dc875f4af" />


#### 2.4.  Integrate modifications in the main table
Once the data has been corrected in administrative_unit_area_<n>_w, it can be integrated in the official table in the au schema.
```
python3 script/integrate.py -c conf.json -T au -t administrative_unit_area_<n>
```

## Step 3: administrative_hierarchy table
The administrative_hierarchy table is a table without geometry which describes the administrative system of each country. This table is an extract of EuroBoundaryMaps' EBM_ISN table for the countries included in the HVLSP.

When a new country is added to the HVLSP, the relevant rows need to be retrieved from EBM_ISN and integrated into au.administrative_hierarchy.
This is currently done with an FME workbench: [AU_manage_administrative_hierarchy.fmw](https://github.com/openmapsforeurope2/fme_workbenches/tree/main/AU). The path to the latest EBM file gdb (locally on the user's computer), the connection information to the HVLSP database and a list of countries to integrate need to be provided.

WARNINGS:
- In the final infrastructure, it will probably not be possible to connect FME directly to the HVLSP database.
- This workbench does not handle updates, it only adds rows to the administrative_hierarchy table but does not update existing rows if the processed country had already been included before.
Therefore, a more permanent solution needs to be determined: cf.issue #16
