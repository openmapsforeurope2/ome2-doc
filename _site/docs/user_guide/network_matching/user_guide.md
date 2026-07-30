# User guide for the production of network data types

## Cleaning
```
$ cd path/to/data-tools/
$ python3 clean.py -c path/to/conf.json -b fr -T tn -t road_link es
```

## Matching
### Data preparation

Prepare tn theme classes (road_link, railway_link) working tables :
```
$ python3 script/prepare_data.py -c config/conf.json -T tn -s 20250908 -m es fr
```

The resulting tables are named as follow:
- <working_schema>.road_link_w_es_fr_20250908
- <working_schema>.railway_link_w_es_fr_20250908

### Matching Processing

Build docker image:
```
$ cd path/to/net_matching/
$ git pull
$ git checkout release/v<version>
$ cd docker
$ ./build-docker-image.sh
```

Run docker container:
```
$ docker run -it net_matching:<version> bash
```

Set the configuration files:
```
& nano config/tn_theme_parameters.ini
& nano config/ra_theme_parameters.ini
```

- check if the parameter DB_CONF_FILE is pointing to the right db configuration file
- set the parameter EDGE_TABLE_INIT (road_link_w_es_fr_20250908, railway_link_w_es_fr_20250908)
- add configuration section for the border to treat if it doesn't exist:
Add the following lines in the configuration files:
```
[es#fr]
COUNTRY_CODE_W  =es#fr
```
If other parameters have to be specialized for this border they must be specified in this section.

Launch matching process:
```
$ ./bin/net_matching --c /usr/local/src/net_matching/config/epg_parameters.ini --T tn --cc es#fr
$ ./bin/net_matching --c /usr/local/src/net_matching/config/epg_parameters.ini --T ra --cc es#fr
```

### Validation preparation

```
$ cd path/to/data-tools/
$ python3 ./script/prepare_data.py -c path/to/conf.json -T tn -s 20250908 -w es fr
```
These command prepare all theme classes for validation. An option -t can be added to prepare data of a single classe (ex: -t road_link)


### Manual Correction

### Integration of matched data in production tables

```
$ cd path/to/data-tools/
$ python3 ./script/integrate_from_validation.py -c path/to/conf.json -T tn es fr
```
These command integrate data of all theme classes. An option -t can be added to integrate data of a specific class (ex: -t road_link)