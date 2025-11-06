# Introduction

This documentation, intended for developers, aims to present details on how the process of harmonizing area features at borders works, as well as the technical principles and configuration of the tool.

# Installation

## Source Code 

The application's source code is available in the [net_area_matching](https://github.com/openmapsforeurope2/net_area_matching.git) repository.

## Dependencies 

Installing the application requires compiling internal and external libraries, including those from IGN.

Here is the dependency graph:

<img width="596" height="593" alt="image" src="https://github.com/user-attachments/assets/fa02cce1-75c0-4546-9b14-88ac96516e04" />

### IGN Socle 

IGN's software core brings together a set of internally developed libraries that unify access to C++ libraries for processing and storing geographic data.  
It provides, among other things, pivot data models (geometries, attribute objects), reading/writing functions for object containers, geometry operations, many algorithms, etc.

The source code for the socle is available in the [sd-socle](http://gitlab.forge-idi.ign.fr/socle/sd-socle.git) repository.

### LibEPG 

This library, developed at IGN and mainly relying on the software socle, contains many algorithms and utility functions specifically dedicated to OME2 production needs.  
It essentially includes generalization functions, utilities for process management such as logging, orchestration, context management).  
It also provides operators for encapsulating complex geometric objects to optimize their manipulation (using graphs, indexes, etc.), thus improving processing performance.

The source code for the libepg library is available at [libepg](http://gitlab.dockerforge.ign.fr/europe/libepg.git).

# Configuration

The tool relies on many configuration parameters to adapt algorithm behavior based on national specificities (semantics, precision, scale, conventions, etc.).

In the [configuration folder](https://github.com/openmapsforeurope2/net_area_matching/tree/main/config), you will find the following files:

- epg_parameters.ini: gathers basic parameters from the libepg library, which forms the development core of the tool. This file also acts as a master file, pointing to additional configuration files.
- db_conf.ini: database connection information.
- theme_parameters.ini: configuration of parameters specific to the application.

# Process Functionality

The harmonization process for area features is launched for a pair of neighboring countries.

## Preliminary Steps

The data on which this process is launched must first be cleaned using the **clean** tool from the [data-tools](https://github.com/openmapsforeurope2/data-tools) project, which removes features outside the country or that are too far from the border.  
This tool should be used on working tables containing extracted data from both countries in the region around their common border.

## General Principle of the Process

The area feature harmonization process is broken down into a series of key steps.
To orchestrate the sequence of these steps, the application uses the **epg::step::StepSuite** tool from the **libepg** library. This allows running a succession of **epg::step::Step** operations, each representing a transformation or process step.
Each step is assigned a code (a three-digit number). Steps are numbered in order. If a step transforms the data it works on, a new table is created for the result, named with the step code as a prefix.  
This ensures that intermediate results are kept, allowing the process to be interrupted and resumed, and facilitating debugging.

The main steps are as follows:

**301** - import 'standing water' objects into the working table  
**310** - generation of 'cutting lines'  
**320** - removal of areas outside their country  
**330** - cleanup of orphan 'cutting lines'  
**334** - creation of intersection areas between both countries, stored in a dedicated table  
**335** - generation of 'cutting points'  
**340** - merging of overlapping areas between both countries  
**350** - splitting of merged areas using 'cutting lines' and sections from 'cutting points'  
**360** - assignment of attributes to the merged/split areas  
**370** - aggregation of merged/split areas  
**399** - export of 'standing water' areas

The **epg::step::StepSuite** tool allows running only selected steps or a range of steps.

## Running the Process

Example: run the complete process for France (country code 'fr') and Belgium (country code 'be'):
```
net_area_matching --c path/to/config/epg_parameters.ini --cc be#fr
```
Note: the `--cc` parameter specifies the code for the border between the two countries. The country code is always composed by concatenating the two country codes separated by a '#'.

Example: run a single step:
```
net_area_matching --c path/to/config/epg_parameters.ini --cc be#fr --sp 340
```

Example: run a range of steps:
```
net_area_matching --c path/to/config/epg_parameters.ini --cc be#fr --sp 340-370
```
Here, all steps from 340 to 370 (inclusive) will be executed.

## Step-by-Step Details

### 301: AddStandingWater

'Standing water' objects from both countries' working tables are copied into the 'watercourse areas' working table. The copied objects are removed from the 'standing water' table.

<img width="809" height="600" alt="image" src="https://github.com/user-attachments/assets/8f5b2230-4183-45ef-84a0-31d97c0e7fd5" />


#### Working Data:

| table                          | input | output | working entity | description                                                 |
|--------------------------------|-------|--------|----------------|-------------------------------------------------------------|
| AREA_TABLE_INIT                | X     | X      | X              | areas to process                                            |
| AREA_TABLE_INIT_STANDING_WATER | X     | X      | X              | areas to export into AREA_TABLE_INIT                        |

#### Main Calculation Operators Used:
- app::calcul::StandingWaterOp

#### Processing Description:
Parameters used:
| parameter              | description                                                          |
|------------------------|----------------------------------------------------------------------|
| IS_STANDING_WATER_NAME | name of the field indicating if the object is 'standing water'       |

A _IS_STANDING_WATER_NAME_ column of type _character varying(255)_ is added to _AREA_TABLE_INIT_ and filled for each imported object.  
The process removes imported objects from _AREA_TABLE_INIT_STANDING_WATER_.

### 310: GenerateCuttingLines

Objective: generate _'cutting lines'_, i.e., portions of contours shared by two polygons of the same country.

#### Working Data:

| table           | input | output | working entity | description                  |
|-----------------|-------|--------|----------------|------------------------------|
| AREA_TABLE_INIT | X     | X      | X              | areas to process             |
| CUTL_TABLE      |       | X      |                | table of 'cutting lines'     |

Note: the _CUTL_TABLE_ output is not prefixed with the step number (it serves as a reference for the whole process).

CUTL_TABLE is dropped and recreated if it exists. Structure:

| field             | type                   |
|-------------------|------------------------|
| COUNTRY_CODE      | character varying(8)   |
| LINKED_FEATURE_ID | character varying(255) |
| GEOM              | LineStringZ            |

#### Main Calculation Operators Used:
- app::calcul::StandingWaterOp

#### Processing Description:
Processing is done country by country. The principle is to build a planar graph from all polygon contours of a country, then identify arcs shared by two surfaces.

<img width="775" height="600" alt="image" src="https://github.com/user-attachments/assets/a01fd3e2-fa6a-4879-80aa-bee13b62deac" />

### 320: CleanByLandmask

Objective: remove areas and area parts too far from their country.

#### Working Data:

| table                 | input | output | working entity | description                   |
|-----------------------|-------|--------|----------------|-------------------------------|
| AREA_TABLE_INIT       | X     | X      | X              | areas to process              |
| TARGET_BOUNDARY_TABLE | X     |        |                | borders                       |
| LANDMASK_TABLE        | X     |        |                | national landmasks            |

#### Main Calculation Operators Used:
- app::calcul::PolygonSplitterOp
- app::calcul::PolygonCleanerOp
- app::calcul::PolygonMergerOp

#### Processing Description:
##### 1) Area Splitting

Parameters used:
| parameter              | description                                           |
|------------------------|-------------------------------------------------------|
| LAND_COVER_TYPE_NAME   | name of the land cover type field                     |
| LAND_COVER_TYPE_VALUE  | value for the land cover type                         |
| PS_BORDER_OFFSET       | offset distance for cutting geometry from the border  |

Areas extending beyond their country are cut along lines corresponding to the international borders, offset by a certain distance outside the country.

<img width="647" height="600" alt="image" src="https://github.com/user-attachments/assets/a353158f-3c46-4cbf-ab52-58ecb2ff294a" />


To optimize calculations, the cutting geometry contouring the country is indexed in the quadtree of the **epg::tools::geometry::SegmentIndexedGeometryCollection** class.

##### 2) Removal of Areas Outside the Country

Parameters used:
| parameter              | description                                           |
|------------------------|-------------------------------------------------------|
| LAND_COVER_TYPE_NAME   | name of the land cover type field                     |
| LAND_COVER_TYPE_VALUE  | value for the land cover type                         |
| PC_DISTANCE_THRESHOLD  | minimum distance threshold for removing areas         |

Areas outside their country and further than a certain threshold are removed.

<img width="643" height="600" alt="image" src="https://github.com/user-attachments/assets/5627e3a4-daaa-430f-87d1-a1b79557266c" />

For optimization, a working area corresponding to the border zone of the country is precomputed as the intersection of a buffer and the national landmask.

_Note: the buffer radius (work zone depth) must be greater than or equal to the extraction distance (data-tools::border_extract parameter)._

##### 3) Merging Split Areas

Parameters used:
| parameter                | description                              |
|--------------------------|------------------------------------------|
| NATIONAL_IDENTIFIER_NAME | Field name for the national identifier   |

Previously split areas that were not removed are merged. This is done by merging all areas with the same value for _NATIONAL_IDENTIFIER_NAME_.

<img width="643" height="600" alt="image" src="https://github.com/user-attachments/assets/3664e814-dca4-41aa-b6e2-8e5dfaeeb35e" />


Notes:
- To avoid creating thin gaps, all merges are preceded by a snapping step (using the ign::geometry::algorithm::SnapOpGeos::SnapTo function).
- Merged areas are cleaned by removing cut points artificially created by splitting/merging actions. Cut points can be identified by the value of their field.

### 330: CleanCuttingLines

This step removes 'cutting lines' that, following the cleaning in the previous step, do not have at least one adjacent area belonging to the same country.

#### Working Data:

| table           | input | output | working entity      | description                  |
|-----------------|-------|--------|---------------------|------------------------------|
| AREA_TABLE_INIT | X     |        |                     | Table of areas to process    |
| CUTL_TABLE      | X     | X      |                     | Table of 'cutting lines'     |

#### Main Calculation Operators Used:
- app::calcul::CuttingLineCleanerOp

#### Processing Description:

Parameters used:
| parameter                | description                                                                           |
|--------------------------|---------------------------------------------------------------------------------------|
| LINKED_FEATURE_ID        | Field containing national identifiers of areas adjacent to the 'cutting line'         |
| NATIONAL_IDENTIFIER_NAME | Field name for the national identifier                                                |

The calculation operator goes through all _'cutting lines'_. For each, it checks if at least one of its two adjacent areas (whose identifiers are concatenated in the _LINKED_FEATURE_ID_ field) belongs to the country. If not, the line is removed.

### 334: GenerateIntersectionAreas

This step generates areas representing the overlap zones between polygons from the two countries.

#### Working Data:

| table                   | input | output | working entity      | description                         |
|-------------------------|-------|--------|---------------------|-------------------------------------|
| AREA_TABLE_INIT         | X     |        |                     | Table of areas to process           |
| INTERSECTION_AREA_TABLE |       | X      |                     | Table of overlapping areas          |

Note: The output table _INTERSECTION_AREA_TABLE_ is not prefixed with the step number (it serves as a reference for the whole process).

The _INTERSECTION_AREA_TABLE_ table, where the overlapping areas are recorded, is deleted if it already exists and then created.
Its structure is as follows:

| field             | type                   |
|-------------------|------------------------|
| COUNTRY_CODE      | character varying(8)   |
| LINKED_FEATURE_ID | character varying(255) |
| GEOM              | MultiPolygonZ          |

#### Main Calculation Operators Used:
- app::calcul::GenerateIntersectionAreaOp

#### Processing Description:

Parameters used:
| parameter                | description                                                                                        |
|--------------------------|----------------------------------------------------------------------------------------------------|
| LINKED_FEATURE_ID        | Field containing the national identifiers of the source areas of the overlapping area              |
| NATIONAL_IDENTIFIER_NAME | Field name for the national identifier                                                             |

The operator iterates through all areas in the _AREA_TABLE_INIT_ table belonging to 'country 1' and calculates the overlap areas with each area in this table belonging to 'country 2'. Each overlap area is stored in the _INTERSECTION_AREA_TABLE_.

<img width="636" height="600" alt="image" src="https://github.com/user-attachments/assets/6e4cd7b4-a5ef-4028-93f9-d9fb13c13382" />

### 335: GenerateCuttingPoints

This step handles the generation of _'cutting points'_. These points will later be used as references to build sections for splitting areas.

#### Working Data:

| table                   | input | output | working entity      | description                        |
|-------------------------|-------|--------|---------------------|------------------------------------|
| AREA_TABLE_INIT         | X     |        |                     | Table of areas to process          |
| INTERSECTION_AREA_TABLE | X     |        |                     | Table of overlapping areas         |
| CUTL_TABLE              | X     |        |                     | Table of 'cutting lines'           |
| CUTP_TABLE              |       | X      |                     | Table of 'cutting points'          |

Note: The output table _CUTP_TABLE_ is not prefixed with the step number (it serves as a reference for the whole process).

The _CUTP_TABLE_ table, where the overlap areas are recorded, is deleted if it already exists and then created.
Its structure is as follows:

| field             | type                   |
|-------------------|------------------------|
| COUNTRY_CODE      | character varying(8)   |
| LINKED_FEATURE_ID | character varying(255) |
| GEOM              | PointZ                 |
| CUTP_SECTION_GEOM | LineString             |

#### Main Calculation Operators Used:
- app::calcul::GenerateCuttingPointsOp

#### Processing Description:

Parameters used:
| parameter                | description                                                                                        |
|--------------------------|----------------------------------------------------------------------------------------------------|
| LINKED_FEATURE_ID        | Field containing the national identifiers of the source areas of the overlap                       |
| NATIONAL_IDENTIFIER_NAME | Field name for the national identifier                                                             |
| DIST_SNAP_MERGE_CF       | Minimum distance between a 'cutting point' and a 'cutting line'                                   |
| CUTP_SECTION_GEOM        | Field name to store the splitting section                                                         |

The calculation of _'cutting points'_ consists of iterating through the areas in _AREA_TABLE_INIT_ for each of the two neighboring countries. For each area, it is determined if a splitting point should be created at the end of a median axis.

The same calculation is performed by iterating through the _INTERSECTION_AREA_TABLE_.

<img width="550" height="813" alt="image" src="https://github.com/user-attachments/assets/fea182a1-bd49-4ca7-aef3-a4b9b58183e1" />


### 340: MergeAreas

This step concerns merging the areas of the two neighboring countries.

#### Working Data:

| table                   | input | output | working entity      | description                        |
|-------------------------|-------|--------|---------------------|------------------------------------|
| AREA_TABLE_INIT         | X     | X      | X                   | Table of areas to process          |

#### Main Calculation Operators Used:
- app::calcul::IntersectingAreasMergerOp

#### Processing Description:

The merging process takes place in two steps:

1. Lists of identifiers are generated for groups of areas to be merged. To form a group, start from a polygon of country 1 and search for all polygons of country 2 that overlap it. The process is repeated until all groups are formed.

<img width="600" height="329" alt="image" src="https://github.com/user-attachments/assets/9bb4f6cb-ad54-4d29-a954-5ab50ed0d8ca" />


2. For each group, the polygons it contains are merged. Only one polygon should result from merging a group since the polygons constitute a single continuous area.

<img width="2056" height="554" alt="image" src="https://github.com/user-attachments/assets/fb2ac02e-4058-4af3-86e7-b774a9f38e3a" />


### 350: SplitMergedAreasWithCF

During this step, the previously merged areas are split according to the _'cutting features'_ (_'cutting points'_ and _'cutting lines'_).

#### Working Data:

| table                   | input | output | working entity      | description                        |
|-------------------------|-------|--------|---------------------|------------------------------------|
| AREA_TABLE_INIT         | X     | X      | X                   | Table of areas to process          |
| CUTL_TABLE              | X     |        |                     | Table of 'cutting lines'           |
| CUTP_TABLE              | X     |        |                     | Table of 'cutting points'          |

#### Main Calculation Operators Used:
- app::calcul::CfSplitterOp

#### Processing Description:

Parameters used:
| parameter                | description                                         |
|--------------------------|-----------------------------------------------------|
| DIST_SNAP_MERGE_CF       | Minimum distance between 'cutting points'           |

To calculate all splitting sections, the areas in the _AREA_TABLE_INIT_ table are iterated. For each area, the first step is to collect all 'cutting points' located on the external contour of the polygon as well as the ends of 'cutting lines' linked to this polygon.

<img width="600" height="641" alt="image" src="https://github.com/user-attachments/assets/32463e48-5c4c-4d79-8356-7b9269205a20" />


The principle for calculating splitting sections from _'cutting points'_ is to project the cutting points onto nearby sub-contours. If it is not possible to select a segment, the point is ignored.
If a _'cutting point'_ is close enough to another _'cutting point'_ (at a distance less than _DIST_SNAP_MERGE_CF_) for which a splitting section has already been calculated, it is ignored.

<img width="571" height="600" alt="image" src="https://github.com/user-attachments/assets/07680c40-a022-4f4c-b3a7-be45629ee997" />


For each [cutting point–projection] section, the proportion of the segment inside the area is calculated. If the ratio is close to zero (almost entirely outside the area), the section is ignored.
If the path is long, the section is considered illegitimate and ignored.

Note: For path calculation, the **epg::tools::MultiLineStringTool** operator is used, which encapsulates the polygon's contour as a simple adjacency graph.

<img width="1182" height="600" alt="image" src="https://github.com/user-attachments/assets/27540e7f-f7e3-4a9f-9f80-da1f8b2de3c9" />


The splitting geometries defined by _'cutting lines'_ often need to be extended at their ends to reach the edges of the merged areas. The axial projection is performed for this purpose.

<img width="849" height="600" alt="image" src="https://github.com/user-attachments/assets/a3d61ed2-59a0-45f7-8fda-2dbd2ac61a0e" />


Once the splitting section is calculated, to avoid precision issues and ensure the split will be performed correctly, both ends of this section are slightly extended.
When several _'cutting lines'_ originally meet at one end, it is ensured that, for this end, the _'cutting lines'_ are projected onto the same point.

<img width="600" height="382" alt="image" src="https://github.com/user-attachments/assets/d33c0292-63a7-44a2-98a0-5571576ddb88" />


Once all splitting geometries calculated from _'cutting lines'_ and _'cutting points'_ are ready, the **app::tools::geometry::PolygonSplitter** operator is used to split the area.

### 360: MergedAttributesAreas

After merging the areas of the two neighboring countries, and then splitting these merged areas by the _'cutting features'_, this step assigns attributes to these new areas.

#### Working Data:

| table                   | input | output | working entity      | description                                                                     |
|-------------------------|-------|--------|---------------------|---------------------------------------------------------------------------------|
| AREA_TABLE_INIT         | X     | X      | X                   | Table of areas to process                                                       |
| AREA_TABLE_INIT_CLEANED | X     |        |                     | Reference table from the **app::step::CleanByLandmask** step                     |

#### Main Calculation Operators Used:
- app::calcul::SetAttributeMergedAreasOp

#### Processing Description:

Parameters used:
| parameter           | description                                                                      |
|---------------------|----------------------------------------------------------------------------------|
| AM_LIST_ATTR_W      | List of working attributes not to be merged                                      |
| AM_LIST_ATTR_JSON   | List of attributes of type json                                                  |
| W_TAG_NAME          | Working field to mark areas resulting from merging                               |

To establish a link between the new areas resulting from the merge/split and the original area objects, for each of them we determine which source object from country A or B it is closest to.
If area A represents less than 10% of area B, the merged area takes B's attributes. Conversely, if B represents less than 10% of A, the merged area takes A's attributes.

As merged/split areas are processed, the _W_TAG_NAME_ field is filled in to keep track of areas resulting from the merge. This information may otherwise be lost at this stage.

### 370: MergeSplitAreas

This step aims to aggregate as much as possible the multitude of areas generated by the merge/split process.

#### Working Data:

| table                   | input | output | working entity      | description                                                                     |
|-------------------------|-------|--------|---------------------|---------------------------------------------------------------------------------|
| AREA_TABLE_INIT         | X     | X      | X                   | Table of areas to process                                                       |

#### Main Calculation Operators Used:
- app::calcul::SplitAreaMergerOp

#### Processing Description:

Parameters used:
| parameter                       | description                                                                |
|----------------------------------|----------------------------------------------------------------------------|
| SAM_SMALL_AREA_THRESHOLD        | Threshold below which an area is considered small                           |
| SAM_SMALL_AREA_LENGTH_THRESHOLD | Length threshold for a small area (length of the median axis)               |
| NATIONAL_IDENTIFIER_NAME        | Field name for the national identifier                                      |
| W_TAG_NAME                      | Working field here used to mark areas resulting from merging                |

All processing described below applies only to areas resulting from merging surfaces from the two neighboring countries (objects with a non-null _W_TAG_NAME_ field).

The first step is to establish a list of small areas ordered by their area (ordering ensures repeatability of aggregations). Only areas meeting the small area criteria are added.

Secondly, an attempt is made to form groups of objects to be merged. For this, small areas are processed in descending order of area, and for each, its largest neighboring area with the same _W_TAG_NAME_ value is sought.
Note that an area '#' (with a double country code) can only be merged with another area '#'.

Groups of objects to be merged are further formed by processing non-small areas and associating them with their largest neighbor with the same _W_TAG_NAME_ value.

Lastly, for each group, all constituent areas are merged. The resulting object inherits the attributes of the largest area in the group.

This entire process is repeated until no more merges can be performed.

<img width="1254" height="400" alt="image" src="https://github.com/user-attachments/assets/e4243b7c-3d7f-4140-9338-bcefb594ee12" />


### 399: SortingStandingWater

In this step, areas from the _'watercourse areas'_ working table marked as _'standing waters'_ are exported for import into the _'standing waters'_ working table.

#### Working Data:

| table                          | input | output | working entity      | description                                                 |
|--------------------------------|-------|--------|---------------------|-------------------------------------------------------------|
| AREA_TABLE_INIT                | X     | X      | X                   | Table of areas to export                                    |
| AREA_TABLE_INIT_STANDING_WATER | X     | X      | X                   | Target table to import areas                                |

#### Main Calculation Operators Used:
- app::calcul::StandingWaterOp

#### Processing Description:

Parameters used:
| parameter                | description                                                 |
|--------------------------|-------------------------------------------------------------|
| IS_STANDING_WATER_NAME   | Field name indicating if the object is 'standing water'     |

All objects for which the _IS_STANDING_WATER_NAME_ field value is as originally set when importing from _AREA_TABLE_INIT_STANDING_WATER_ are exported from the _AREA_TABLE_INIT_ table and imported into _AREA_TABLE_INIT_STANDING_WATER_.

# Glossary

- **Cutting line**: Arc representing the section of contour shared between two adjacent surfaces of the same country. This arc is used for area splitting.
- **Cutting point**: Specific point on the contour of an area located at the end of the median axis. This point is used as a reference to create a splitting section.
