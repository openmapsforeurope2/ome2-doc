# Matching


## Hydrography theme (HY)

### Data preparation

Tool : data-tools

Edit configuration file :

```
"db_conf_file":<db_conf_file_name>.json,
```

Run preparation process :

```
python3 prepare_data.py -c path/to/conf.json -T hy -s <suffix> -m <country_code> <country_code>
```

Create tables:
- watercourse_area_w_<country_code>_<country_code>_<suffix>
- standing_water_w_<country_code>_<country_code>_<suffix>
- watercourse_link_w_<country_code>_<country_code>_<suffix>

### Area Matching

Tool : area_matching

Edit configuration file theme_parameters.ini :

```
AREA_TABLE_INIT                       =watercourse_area_w_<country_code>_<country_code>_<suffix>
AREA_TABLE_INIT_STANDING_WATER        =standing_water_w_<country_code>_<country_code>_<suffix>
```

Add specific border configuration if it does not exist:

```
[<country_code>#<country_code>]
COUNTRY_CODE_W                        =<country_code>#<country_code>
```

Specify additional parameters in this section if necessary


Run matching process :

```
area_matching --c path/to/epg_parameters.ini --cc <country_code>#<country_code>
```


### Network Matching

Tool : net_matching

Edit configuration file hy_theme_parameters.ini :

```
EDGE_TABLE_INIT                           =watercourse_link_w_<country_code>_<country_code>_<suffix>
WATERCOURSE_AREA_TABLE                    =_399_watercourse_area_w_<country_code>_<country_code>_<suffix>
STANDING_WATER_TABLE                      =_399_standing_water_w_<country_code>_<country_code>_<suffix>
```

Add specific border configuration if it does not exist:

```
[<country_code>#<country_code>]
COUNTRY_CODE_W                        =<country_code>#<country_code>
```

Specify additional parameters in this section if necessary

Run matching process :

```
net_matching --c path/to/epg_parameters.ini --T hy --cc <country_code>#<country_code>
```

### Prepare data for validation

Tool : data-tools

```
python3 prepare_data.py -c path/to/conf.json -T hy -s <suffix> -w <country_code> <country_code>
```


### Integrate data after validation data for validation

Tool : data-tools

```
python3 integrate_from_validation.py -c path/to/conf.json -T hy <country_code> <country_code>
```