# data-tools



## Description

The data-tools utility tools based on SQL scripts to manage the central OME2 database.

The tools provided are as follows:

- `create_table`: generates and runs the scripts to create all the tables required of the OME2 central large-scale database.
- `create_view`: generates and runs the scripts to create the release views of the OME2 central large-scale database.
- `border_extract`: used to extract objects around an international boundary from a source table to a target work table for further processing.
- `clean`: removes data outside a country's extent (starting from a distance threshold). This cleanup is the first step of the data harmonization process along borders. This function includes extraction, cleaning, and integration steps.
- `integrate`: reintegrates into the source table the data extracted and processed in the work table.
- `prepare_data`: prepares the data required for the edge-matching process or validation phase (after the edge-matching).
- `integrate_from_validation`: updates the production tables by integrating changes from the validation tables (initial table and processed data table).
- `copy_table`: copies tables located in a schema into the public schema.
- `revert`: /!\ DEPRECATED - undoes the changes corresponding to the 'step' specified as a parameter. All changes linked to subsequent 'steps' are also undone.


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

**Purpose**

This function creates a table or all of a theme's tables, for example when a new OME2 database is set up.

**Parameters**
* c [mandatory]: configuration file
* T [mandatory]: theme (only one theme can be specified)
* t [optional]: table (multiple tables can be specified by repeating this option as necessary). Tables must belong to theme T

**Examples**

Example for creating all tables for the Transport theme:
~~~
python3 script/create_table.py -c path/to/conf.json -T tn
~~~

Example for creating a single table:
~~~
python3 script/create_table.py -c path/to/conf.json -T tn -t railway_link
~~~

### create_view

**Purpose** 

This function creates one or more views corresponding to a table or to all of a theme's tables in the "release" schema, for example when a new OME2 database is set up.
These views will only contain "live" objects (not destroyed objects i.e. objects for which end_lifespan_version is NULL).

**Parameters**
* c [mandatory]: configuration file
* T [mandatory]: theme (only one theme can be specified)
* t [optional]: table (multiple tables can be specified by repeating this option as necessary). Tables must belong to theme T

**Examples**

Example for creating view corresponding to all tables for the Transport theme:
~~~
python3 script/create_view.py -c path/to/conf.json -T tn
~~~

Example for creating a view corresponding to a single table:
~~~
python3 script/create_view.py -c path/to/conf.json -T tn -t railway_link
~~~

### border_extract

**Purpose**

This function extracts data located in a buffer along one or more international boundaries for one country or for two neighbouring countries. The data is extracted for one table or for a whole theme.

**Parameters**
* c [mandatory]: configuration file
* T [mandatory]: theme (only one theme can be specified)
* t [optional]: table (multiple tables can be specified by repeating this option as necessary). Tables must belong to theme T
* d [mandatory]: buffer radius
* b [optional]: neighbouring country code
* n [optional]: option to prevent deleting data already present in the work table
* arguments: codes of country/countries to extract (one or two)

**Examples**

Example for extracting data from a country along all its borders:
~~~
python3 script/border_extract.py -c path/to/conf.json -T tn -t road_link -d 4000 nl '#'
~~~

Example for extracting data from two neighboring countries:
~~~
python3 script/border_extract.py -c path/to/conf.json -T hy -t watercourse_link -d 1000 be fr
~~~

Example for extracting all data of a country and data from neighboring countries border by border (this can be useful for countries with long boundaries):
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

### clean

**Purpose**

This function removes national data located outside a country's extent (starting from a distance threshold). This cleanup is the first step of the data harmonization process along borders. It includes extraction, cleaning, and integration steps.

**Parameters**
* c [mandatory]: configuration file
* T [mandatory]: theme (only one theme can be specified)
* t [optional]: table (multiple tables can be specified by repeating this option as necessary). Tables must belong to theme T. If not defined, the cleaning will be processed for all tables of the theme (specified with the -T parameter).
* b [optional]: country code for a neighbouring country (multiple codes can be specified by repeating this option as many times as necessary). If this parameter is defined, the cleaning will be processed only on the specified border(s).
* i [optional]: if specified, the cleaning is performed around disputed borders.
* a [optional]: parameter to process cleaning around all borders of the specified country/countries (see "arguments"). If specified, all defined -b parameters will be ignored.
* arguments: codes of country/countries to clean

**Examples**

Example of cleaning French data around the borders with Luxembourg and Belgium:
~~~
python3 script/clean.py -c path/to/conf.json -b lu -b be -T tn -t road_link fr
~~~

Example of cleaning French data around all borders:
~~~
python3 script/clean.py -c path/to/conf.json -a -T tn -t road_link fr
~~~

### integrate

**Purpose**

This function reintegrates the data which was extracted and processed in the work table into the original source table.

**Parameters**
* c [mandatory]: configuration file
* T [mandatory]: theme (only one theme can be specified)
* t [optional]: table (multiple tables can be specified by repeating this option as necessary). Tables must belong to theme T

**Examples**

Example usage:
~~~
python3 script/integrate.py -c path/to/conf.json -T tn -t road_link
~~~

### prepare_data

**Purpose**

This function prepares the data required for the edge-matching process or validation phase (after the edge-matching).

**Parameters**
* c [mandatory]: configuration file
* T [mandatory]: theme (only one theme can be specified)
* t [optional]: table (multiple tables can be specified by repeating this option as necessary). Tables must belong to theme T.
* s [mandatory]: suffix applied for working table naming. Parameter to specify only in case of preparation for matching (see -m option).
* n [optional]: parameter to specify to prepare data for net_matching
* a [optional]: parameter to specify to prepare data for au_matching
* w [optional]: parameter to specify to prepare data for validation
* arguments: codes of two border countries

**Examples**

Example of data preparation for the matching process:
~~~
python3 script/prepare_data.py -c path/to/conf.json -n -T tn -t road_link -s 20250904 be fr
~~~

Example of data preparation for the validation process:
~~~
python3 script/prepare_data.py -c path/to/conf.json -w -T tn -t road_link -s 20250904 be fr
~~~

### integrate_from_validation

**Purpose**

Once the edge-matching has been performed on a theme between two countries, the results are copied into "validation" tables. These tables are manually
reviewed and corrected by data experts. This function updates the source tables by integrating changes from the validation tables.

**Parameters**
* c [mandatory]: configuration file
* T [mandatory]: theme (only one theme can be specified)
* t [optional]: table (multiple tables can be specified by repeating this option as necessary). Tables must belong to theme T.
* arguments: codes of the two matched countries to integrate

**Examples**

Example of updating the production tables for the entire hydrography theme after the data matching process between Austria (at) and Czechoslovakia (cz):
~~~
python3 script/integrate_from_validation.py -c path/to/conf.json -T hy at cz
~~~

### copy_table

**Purpose**

This function allows to copy the schema.table into public.schema_table.

**Parameters**
* c [mandatory]: configuration file
* arguments: table(s) to copy

**Example**

~~~
python3 script/copy_table.py -c path/to/conf.json au.administrative_unit_area_1 ib.international_boundary_line
~~~

### revert
/!\ DEPRECATED

**Purpose**

This function undoes the changes corresponding to the 'step' specified as a parameter. All changes linked to subsequent 'steps' are also undone.

**Parameters**
* c [mandatory]: configuration file
* T [mandatory]: theme (only one theme can be specified)
* t [optional]: table (multiple tables can be specified by repeating this option as necessary). Tables must belong to theme T
* s [mandatory]: step number

**Example**

~~~
python3 script/revert.py -c path/to/conf.json -T au -t administrative_unit_area_3 -s 30
~~~
