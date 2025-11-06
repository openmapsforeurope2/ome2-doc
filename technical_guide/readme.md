# Technical guide

A number of tools have been developed for the OME2 project, in order to ensure the production of the High-Value Large-Scale Prototype. The technical documentation for each tool can be found in its GitHub repository.

## Model conversion tool
* [data-model-transformer](https://github.com/openmapsforeurope2/data-model-transformer) : transform national PostgreSQL databases into the OME2 data model implemented in PostgreSQL (model conversion tool). 

## Edge-matching tools
* [au_matching](https://github.com/openmapsforeurope2/au_matching): align administrative units at the lowest national level with the international boundaries used by the OME2 project (edge-matching tool for administrative units #1). 
* [au_merging](https://github.com/openmapsforeurope2/au_merging): generate level N administrative units from edge-matched level N-1 administrative units (edge-matching tool for administrative units #2).
* [net_matching](https://github.com/openmapsforeurope2/net_matching): edge-match network features from the transport and hydrography themes between two countries along their common international boundary. 
* [net_area_maching](https://github.com/openmapsforeurope2/net_area_matching): edge-match watercourse_area and standing_water from the hydrography theme between two countries along their common international boundary. 
* [net_point_maching](https://github.com/openmapsforeurope2/net_point_matching): ensure the consistency of road and hydro nodes after networks have been edge-matched.
* [area_maching](https://github.com/openmapsforeurope2/area_matching): edge-match other area features from the hydrography theme (drainage_basin, glacier_snowfield) between two countries along their common international boundary. 

## Update tools
* [unmatching](https://github.com/openmapsforeurope2/unmatching)
* [change_detection](https://github.com/openmapsforeurope2/change_detection)
* [up_area_tools](https://github.com/openmapsforeurope2/up_area_tools)

## Utility tools
* [data-tools](https://github.com/openmapsforeurope2/data-tools) : SQL utility tools. 

