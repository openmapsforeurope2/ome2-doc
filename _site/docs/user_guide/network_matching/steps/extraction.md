/!\ TO BE DELETED

# extraction
```
python3 border_extract.py -c conf.json -T <theme> -t <net_type> -d <distance> <country_code> '#'
```
## Extract parameters for clean
<br>

Below are the distance parameters used to extract data for cleaning (i.e. deleting data out of the country's extent).

<strong>net_type = road_link, theme = tn</strong>
| country                        | country_code | distance | 
|--------------------------------|--------------|----------|
| The Netherlands                | nl           | 4000     |
| Belgium                        | be           | 4000     |
| France                         | fr           | 3000     |
| Suisse                         | ch           | 4000     |
| Luxembourg                     | lu           | 4000     |

<br>

<strong>net_type = railway_link, theme = tn</strong>
| country                        | country_code | distance |
|--------------------------------|--------------|----------|
| The Netherlands                | nl           | 4000     |
| Belgium                        | be           | 4000     |
| France                         | fr           | 4000     |
| Suisse                         | ch           | 4000     |
| Luxembourg                     | lu           | 4000     |

<br>

<strong>net_type = watercourse_link, theme = hy</strong>
| country                        | country_code | distance |
|--------------------------------|--------------|----------|
| The Netherlands                | nl           | 4000     |
| Belgium                        | be           | 4000     |
| France                         | fr           | 3000     |
| Suisse                         | ch           | 4000     |
| Luxembourg                     | lu           | 4000     |

<br>

<u>France:</u>

ATTENTION!
voir si pour avoir les zone "in dispute", plutot 
```
<theme> -t <net_type> -B international -d <distance> fr
```

```
python3 script/border_extract.py -c conf.json -T <theme> -t <net_type> -b false -B international -d <distance> fr
python3 script/border_extract.py -c conf.json -T <theme> -t <net_type> -b ad -d <distance> -n fr
python3 script/border_extract.py -c conf.json -T <theme> -t <net_type> -b mc -d <distance> -n fr
python3 script/border_extract.py -c conf.json -T <theme> -t <net_type> -b lu -d <distance> -n fr
python3 script/border_extract.py -c conf.json -T <theme> -t <net_type> -b it -d <distance> -n fr
python3 script/border_extract.py -c conf.json -T <theme> -t <net_type> -b es -d <distance> -n fr
python3 script/border_extract.py -c conf.json -T <theme> -t <net_type> -b ch -d <distance> -n fr
python3 script/border_extract.py -c conf.json -T <theme> -t <net_type> -b de -d <distance> -n fr
python3 script/border_extract.py -c conf.json -T <theme> -t <net_type> -b be -d <distance> -n fr
```

## Extract parameters for edge-matching
<br>

Below are the distance parameters used to extract data for edge-matching between two countries.

TO-DO
