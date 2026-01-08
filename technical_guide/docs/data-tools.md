# data-tools



## Description

The data-tools repository contains all the features of the OME2 project that involve executing SQL scripts to manage the central database.

The tools provided are as follows:

- `create_table`: generates and runs the scripts to create all the tables required of the OME2 central large-scale database.
- `border_extract`: used to extract objects around an international boundary from a source table to a target work table for further processing.
- `integrate`: reintegrates into the source table the data extracted and processed in the work table.
- `revert`: undoes the changes corresponding to the 'step' specified as a parameter. All changes linked to subsequent 'steps' are also undone.
- `copy_table`: copies tables located in a schema into the public schema.
- `clean`: removes data outside a country' extent (starting from a distance threshold). This cleanup is the first step of the data harmonization process along borders. This function includes extraction, cleaning, and integration steps.
- `integrate_from_validation`: updates the production tables by integrating changes from the validation tables (initial table and processed data table).
- `prepare_data`: prepares the data required for the edge-matching or validation (after the edge-matching) processes.

This project also includes [SQL scripts](https://github.com/openmapsforeurope2/data-tools/tree/main/sql/db_init) intended to set up the internal mechanisms of the OME2 database (life-cycle management, resolution, identifiers, etc.).

## Configuration

The configuration of this project describes the data model of the tables and the structure of the OME2 database.

The configuration files are located in the [configuration folder](https://github.com/openmapsforeurope2/data-tools/tree/main/conf) and are as follows:
- `conf.json`: this file lists the tables of each theme, their distribution across different schemas, their main fields (work field, identifier, geometry, country code). This file also specifies the table naming system (suffix for work tables, reference, updates, etc.). This is the base configuration used by all tools. It points to the db_conf.json file.
- `db_conf.json`: database connection information.
- `mcd.json`: this file describes the data model for all the tables. It is only used by the 'create_table' function.

## Usage

All the tools should be used on the OME2 production platform. They can also be used via command lines.

### create_table

This function creates a table or all tables of a theme, for example when a new OME2 database is set up.

Parameters
* c [mandatory]: configuration file
* T [mandatory]: theme (only one theme can be specified)
* t [optional]: table (multiple tables can be specified by repeating this option as necessary). Tables must belong to theme T

<br>

Example for creating all tables for the Transport theme:
~~~
python3 script/create_table.py -c path/to/conf.json -T tn
~~~

Example for creating a single table:
~~~
python3 script/create_table.py -c path/to/conf.json -T tn -t railway_link
~~~

### create_view

This function creates view(s) of a table or all tables of a theme in the "release" schema, for example when a new OME2 database is set up.
These views will only contain "live" objects (not destroyed objects i.e. objects for which end_lifespan_version is NULL).

Parameters
* c [mandatory]: configuration file
* T [mandatory]: theme (only one theme can be specified)
* t [optional]: table (multiple tables can be specified by repeating this option as necessary). Tables must belong to theme T

<br>

Example for creating view corresponding to all tables for the Transport theme:
~~~
python3 script/create_view.py -c path/to/conf.json -T tn
~~~

Example for creating a view corresponding to a single table:
~~~
python3 script/create_view.py -c path/to/conf.json -T tn -t railway_link
~~~

### border_extract

Parameters
* c [mandatory]: configuration file
* T [mandatory]: theme (only one theme can be specified)
* t [optional]: table (multiple tables can be specified by repeating this option as necessary). Tables must belong to theme T
* d [mandatory]: buffer radius
* b [optional]: neighbouring country code
* n [optional]: option to prevent deleting data already present in the work table
* arguments: codes of country/countries to extract (one or two)

<br>

Example for extracting data from a country along all its borders:
~~~
python3 script/border_extract.py -c path/to/conf.json -T tn -t road_link -d 4000 nl '#'
~~~

Example for extracting data from two neighboring countries:
~~~
python3 script/border_extract.py -c path/to/conf.json -T hy -t watercourse_link -d 1000 be fr
~~~

Example for extracting all data of a country and data from neighboring countries border by border:
~~~
python3 script/border_extract.py -c path/to/conf.json -T tn -t road_link -b false -B international -d 3000 fr
python3 script/border_extract.py -c path/to/conf.json -T tn -t road_link -b ad -d 3000 -n fr
python3 script/border_extract.py -c path/to/conf.json -T tn -t road_link -b mc -d 3000 -n fr
python3 script/border_extract.py -c path/to/conf.json -T tn -t road_link -b lu -d 3000 -n fr
python3 script/border_extract.py -c path/to/conf.json -T tn -t road_link -b it -d 3000 -n fr
python3 script/border_extract.py -c path/to/conf.json -T tn -t road_link -b es -d 3000 -n fr
python3 script/border_extract.py -c path/to/conf.json -T tn -t road_link -b ch -d 3000 -n fr
python3 script/border_extract.py -c path/to/conf.json -T tn -t road_link -b de -d 3000 -n fr
python3 script/border_extract.py -c path/to/conf.json -T tn -t road_link -b be -d 3000 -n fr
~~~
> _Note: The first line allows extraction of data around international borders that have a simple country code. This corresponds to disputed borders._

### integrate

Parameters
* c [mandatory]: configuration file
* T [mandatory]: theme (only one theme can be specified)
* t [optional]: table (multiple tables can be specified by repeating this option as necessary). Tables must belong to theme T

<br>

Example usage:
~~~
python3 script/integrate.py -c path/to/conf.json -T tn -t road_link
~~~

### revert

Parameters
* c [mandatory]: configuration file
* T [mandatory]: theme (only one theme can be specified)
* t [optional]: table (multiple tables can be specified by repeating this option as necessary). Tables must belong to theme T
* s [mandatory]: step number

<br>

Example usage:
~~~
python3 script/reverte.py -c path/to/conf.json -T au -t administrative_unit_area_3 -s 30
~~~



### clean

Parameters
* c [mandatory]: configuration file
* T [mandatory]: theme (only one theme can be specified)
* t [optional]: table (multiple tables can be specified by repeating this option as necessary). Tables must belong to theme T. If not defined, the cleaning will be processed for all tables of the theme (specified with the -T parameter).
* b [optional]: country code for a border country (multiple codes can be specified by repeating this option as many times as necessary). If this parameter is defined, the cleaning will be processed only on the specified border(s).
* i [optional]: if specified, the cleaning is performed around disputed borders.
* a [optional]: parameter to process cleaning around all borders of the specified country/countries (see "arguments"). If specified, all defined -b parameters will be ignored.
* arguments: codes of country/countries to clean

<br>

Example of cleaning French data around the borders with Luxembourg and Belgium:
~~~
python3 script/clean.py -c path/to/conf.json -b lu -b be -T tn -t road_link fr
~~~

Example of cleaning French data around all borders:
~~~
python3 script/clean.py -c path/to/conf.json -a -T tn -t road_link fr
~~~

### copy_table

This function allows you to copy the schema.table into public.schema_table.

Parameters
* c [mandatory]: configuration file
* arguments: table(s) to copy

<br>

Example usage:
~~~
python3 script/copy_table.py -c path/to/conf.json au.administrative_unit_area_1 ib.international_boundary_line
~~~

### integrate_from_validation

Parameters
* c [mandatory]: configuration file
* T [mandatory]: theme (only one theme can be specified)
* t [optional]: table (multiple tables can be specified by repeating this option as necessary). Tables must belong to theme T.
* arguments: codes of the two matched countries to integrate

Example of updating the production tables for the entire hydrography theme after the data matching process between Austria (at) and Czechoslovakia (cz):
~~~
python3 script/integrate_from_validation.py -c path/to/conf.json -T hy at cz
~~~

### prepare_data

Parameters
* c [mandatory]: configuration file
* T [mandatory]: theme (only one theme can be specified)
* t [optional]: table (multiple tables can be specified by repeating this option as necessary). Tables must belong to theme T.
* s [mandatory]: suffix applied for working table naming. Parameter to specify only in case of preparation for matching (see -m option).
* n [optional]: parameter to specify to prepare data for net_matching
* a [optional]: parameter to specify to prepare data for au_matching
* w [optional]: parameter to specify to prepare data for validation
* arguments: codes of two border countries

Example of data preparation for the matching process:
~~~
python3 script/prepare_data.py -c path/to/conf.json -n -T tn -t road_link -s 20250904 be fr
~~~

Example of data preparation for the validation process:
~~~
python3 script/prepare_data.py -c path/to/conf.json -w -T tn -t road_link -s 20250904 be fr
~~~

### OME2 Database Creation

Below is the ordered list of commands to run to create the OME2 database.

#### Structure Initialization
~~~
psql -h SMLPOPENMAPS2 -p 5432 -U postgres -d OME2 -f ./sql/db_init/HVLSP_0_GCMS_0_ADMIN.sql
psql -h SMLPOPENMAPS2 -p 5432 -U postgres -d OME2 -f ./sql/db_init/HVLSP_1_CREATE_SCHEMAS.sql
psql -h SMLPOPENMAPS2 -p 5432 -U postgres -d OME2 -f ./sql/db_init/ome2_reduce_precision_3d_trigger_function.sql
psql -h SMLPOPENMAPS2 -p 5432 -U postgres -d OME2 -f ./sql/db_init/ome2_reduce_precision_2d_trigger_function.sql
~~~

#### Table Creation

Create all tables for all schemas:

~~~
python3 script/create_table.py -c path/to/conf.json -m mcd.json -T tn
python3 script/create_table.py -c path/to/conf.json -m mcd.json -T hy
python3 script/create_table.py -c path/to/conf.json -m mcd.json -T au
python3 script/create_table.py -c path/to/conf.json -m mcd.json -T ib
~~~

#### Table History Management
~~~
psql -h SMLPOPENMAPS2 -p 5432 -U postgres -d ome2_test_cd -f ./sql/db_init/HVLSP_2_GCMS_1_COMMON.sql
psql -h SMLPOPENMAPS2 -p 5432 -U postgres -d ome2_test_cd -f ./sql/db_init/HVLSP_3_GCMS_3_HISTORIQUE.sql
psql -h SMLPOPENMAPS2 -p 5432 -U postgres -d ome2_test_cd -f ./sql/db_init/ign_gcms_history_trigger_function.sql
psql -h SMLPOPENMAPS2 -p 5432 -U postgres -d ome2_test_cd -f ./sql/db_init/HVLSP_4_GCMS_4_OME2_ADD_HISTORY.sql
psql -h SMLPOPENMAPS2 -p 5432 -U postgres -d ome2_test_cd -c "ALTER SEQUENCE public.seqnumrec OWNER TO g_ome2_user;"
~~~
