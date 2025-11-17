# User guide for the production of the OME2 high-value large-scale dataset

## User guide to integrate a new country
When data is provided for a new country, here are the main steps to follow:
* Upload the data in a PostGIS database on the OME2 server.
* Convert the data to integrate it in the central database.
* Process international boundaries.
* If neighbouring countries are already included, edge-match the data for each theme successively, starting with Administrative units (AU).

These steps are detailed below.

### 1. Upload to PostGIS

### 2. Model conversion
National producers can either provide:
* national datasets in their own data model: in this case, they are required to provide a mapping table explaining how to transform their data into the OME2 data model.
* INSPIRE datasets: in this case, it is considered that the OME2 team is able to understand the transformation on their own, so no mapping table is required.

This data needs to be converted to the OME2 data model and integrated in the central database, using the [model conversion tool](https://github.com/openmapsforeurope2/data-model-transformer).
The model conversion tool needs one JSON configuration file per theme to run the transformation. Explanations on the implementation of the configuration files and on how to run the tool are available in its [documentation](https://github.com/openmapsforeurope2/data-model-transformer).


### 3. International boundaries
For the OME2 project, in order to harmonise data across countries, it is necessary to use common international boundaries. However, neighbouring countries often provide different geometries for their international boundaries, even when these are officially agreed.

An FME workbench was therefore put into place to calculate common "technical" boundaries, which are considered as a reference for the project: [boundary_unification_process](https://github.com/openmapsforeurope2/fme_workbenches/blob/main/boundary_unification_process/Boundary_unification_process.fmw)

> TO-DO: move to boundary_unification doc
> - When an isolated country is integrated, only the first step of the workbench needs to be used. This creates the boundary lines (including coastlines) all around the country.
> - Then the lines need to be split manually where there is a change in country code or boundary type (international boundary vs coastline).
> - Create point in ib.international_boundary_node when there is a change in the boundary line (type or country code).
> - Use EBM as reference.


### 4. Administrative units theme (AU)

#### 4.1. Edge-matching for the AU theme
[User guide for AU matching](https://github.com/openmapsforeurope2/OME2/blob/main/docs/prod/administrative_unit_area_matching/user_guide_au.md)

#### 4.2. Administrative_hierarchy table
The administrative_hierarchy table is a table without geometry which describes the administrative system of each country. This table is an extract of EuroBoundaryMaps' EBM_ISN table for the countries included in the HVLSP.

When a new country is added to the HVLSP, the relevant rows need to be retrieved from EBM_ISN and integrated into au.administrative_hierarchy.

This is currently done with an FME workbench: [AU_manage_administrative_hierarchy.fmw](https://github.com/openmapsforeurope2/fme_workbenches/blob/main/AU/AU_manage_administrative_hierarchy.fmw)
The path to the latest EBM file gdb (locally on the user's computer), the connection information to the HVLSP database and a list of countries to integrate need to be provided.

> _WARNING:_ 
> - _in the final infrastructure, it will probably not be possible to connect FME directly to the HVLSP database._
> - _this workbench does not handle updates, it only adds rows to the administrative_hierarchy table but does not update existing rows if the processed country had already been included before._
> 
> _Therefore, a more permanent solution needs to be determined: cf.[issue #16](https://github.com/openmapsforeurope2/OME2/issues/16)_

### 5. Transport network theme (TN)
* [Cleaning](https://github.com/openmapsforeurope2/OME2/blob/main/docs/prod/network_matching/steps/cleaning.md)
* [Network matching](https://github.com/openmapsforeurope2/OME2/blob/main/docs/prod/network_matching/steps/matching.md)
* [Integration](https://github.com/openmapsforeurope2/OME2/blob/main/docs/prod/network_matching/steps/integration.md)

### 6. Hydrography theme (HY)
* [Cleaning](https://github.com/openmapsforeurope2/OME2/blob/main/docs/prod/network_matching/steps/cleaning.md)
* [Area matching]
* [Network matching](https://github.com/openmapsforeurope2/OME2/blob/main/docs/prod/network_matching/steps/matching.md)
* [Integration](https://github.com/openmapsforeurope2/OME2/blob/main/docs/prod/network_matching/steps/integration.md)

## User guide to update an existing country
