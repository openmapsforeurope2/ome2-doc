# Data Model Transformer – Technical Documentation


The **data-model-transformer** tool, also known as "Harmonization tool", aims at converting national data provided by producers for their country to the ECRM (ex-HVLSP) data model, then integrates the converted data into the central ECRM database. The following figure summarizes how the harmonization tool works:

<p align="center">
   <img width="681" height="331" alt="image" src="https://github.com/user-attachments/assets/3e0d3b91-c02f-4e9a-88be-93b6f3b8a3d0" />
</p>

## Main characteristics of the tool
### Repository structure
The source code repository contains the following folders:
- config: JSON configuration files used by the tool (see §3.2.2);
- docker: dockerfile and build script for reproducible setup;
- functions: functions implemented to operate some complex transformations (see §3.2.2);
- script: orchestrating script and additional script, which constitute the tool.
Additional files such as README, VERSION, LICENSE, CHANGELOG are also included.

### License
The Harmonization tool is distributed under the MIT license.

### Technical characteristics
The Harmonization tool has been developed with Python 3.11. It uses standard Python libraries to handle PostGIS databases.
The Harmonization tool may be run from Windows or Linux environments.

### Deployment
The Harmonization tool is deployed on a Cloud infrastructure hosted by OVH-Cloud, which is often referred to as the “OME2 production platform”. This infrastructure was set into place and is maintained by IGN-France. Its access is restricted to a list of authorized IP addresses for security reasons.

### Running the tool
The Harmonization tool may be run from the OME2 production platform where a simple Jenkins interface is available to the OME2 producers. The interface enables the producer to specify a number of parameters:
-	The name of the source national database (a PostGIS database stored on the production platform);
-	The source PostGIS schema name in the national database;
-	The target PostGIS schema name in the HVLSP central database;
-	The country code of the country to process;
-	The theme to process (au = Administrative units, tn = Transport network, hy = Hydrography).

<p align="center">
   <img width="509" height="321" alt="image" src="https://github.com/user-attachments/assets/2d0a7afe-fe65-4b97-88de-f8a17d977055" />
</p>
Once the parameters are filled, the tool can be run by clicking on the “Build” button.

### Demonstration
A video which demonstrates how to use the tool and its results is available via the following [link](https://github.com/openmapsforeurope2/tools_demo/tree/main/D5.1_Harmonization_tool). 


## Tool description
### Input
#### PostGIS national database
The Harmonization tool uses as input a national PostGIS database. 
During the OME2 project, national producers provided their data in varying formats: GeoPackage, gml, SQL dump, Shapefile, Geodatabase. It was up to the WP4 team to first upload these datasets on the OME2 PostGIS server, hosted on the production platform. Several methodologies and software solutions were used, depending on the data provided (FME, SQL restore…).
On the production platform, a PostGIS database was created for each country, with the following naming convention: <country code>_<delivery date> (e.g. at_20250120).
<p align="center">
   <img width="172" height="183" alt="image" src="https://github.com/user-attachments/assets/a5fbc6bd-fd30-4885-a619-232e0c57f3c8" />
</p>

#### Configuration files
The Harmonization tool also needs “configuration files” to run. These are JSON files which describe how to transform the national data model into the HVLSP data model. They are based on mapping tables (Excel spreadsheet) which are filled by national producers to explain which tables, attributes and values from their datasets should be used to fill the HVLSP information.
<p align="center">
   <img width="628" height="217" alt="image" src="https://github.com/user-attachments/assets/426fab06-8de1-4dc3-bc8c-1570799e8b40" />
</p>
The JSON files are implemented based on the information provided. The example below shows for instance that:
-	The target table “aerodrome_point” should be filled with information contained in the national table “aerodrome”;
-	Not all objects from the source table should be used, as indicated by the selection query (“where” clause);
-	Some source attributes can be directly used to fill target attributes with a simple 1-1 mapping, e.g. “w_national_identifier”;
-	Some attributes can be mapped with a simple test or a simple value, such as “designator_iata” or “xy_source”;
-	Other require a slightly more complex mapping, for instance using several source attributes. In such cases, a simple Python function can be written, for instance for “aerodrome_category”.
<p align="center">
   <img width="761" height="405" alt="image" src="https://github.com/user-attachments/assets/a9f51c59-3a04-4575-beb3-69607a3ba913" />
</p>
There is one JSON configuration per country and per theme. A naming convention is applied: <country_code>-<theme>.json. When the tool is run (see §2.5), the “COUNTRY” and “THEME” parameters are used to determine the name of the configuration file to be used.
For example, if COUNTRY = ‘be’ and THEME = ‘hy’, the configuration file will be “be-hy.json”.
The configuration files, implemented for the 10 countries already included in the prototype, are stored in the repository’ [config](https://github.com/openmapsforeurope2/data-model-transformer/tree/main/config) folder. The functions which were implemented are stored in the [functions](https://github.com/openmapsforeurope2/data-model-transformer/tree/main/functions) folder.

### Key features
The main Python scripts which constitute the Harmonization tool are stored in the repository’s [script](https://github.com/openmapsforeurope2/data-model-transformer/tree/main/script) folder:
- **transform.py** is the “orchestrator” or main script: it is launched when the tool is run, then successively calls the following three scripts.
- **extract.py** extracts the relevant data from the source database according to the JSON configuration file. Only the necessary tables and fields are extracted, and only the necessary objects if selection queries are specified in the configuration file. If necessary, geometries are projected into the coordinate reference system used for the production of the HVLSP (ETRS89 Lambert Azimuthal Area – EPSG:3035) using PostgreSQL functions. The extracted data is stored in temporary json files. 
- **dump.py** creates sql dump files using the data extracted during the previous step. The transformations indicated in the configuration files are applied to each attribute to calculate the values to insert. The dump files are stored in an output folder.
<p align="center">
   <img width="1015" height="106" alt="image" src="https://github.com/user-attachments/assets/7c571196-096a-4ebd-bba2-6cdcb5460f0b" />
</p>
- **restore.py** runs the dump files created with the previous script in the target HVLSP database. As a result, the transform data is directly integrated with the right structure in the target tables.

### Output
As a result, the transformed data is directly integrated into the central HVLSP database. According to options chosen by the user, it can either replace or enrich the data already contained in the database for the processed country and theme.
The Harmonization tool needs to be run for each country and each theme separately to fill the database.
