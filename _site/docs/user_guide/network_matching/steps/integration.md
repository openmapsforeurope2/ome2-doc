### integration

#### General command line:
The integration command line will integrate the data from the work table (e.g. road_link_w) into the original table (e.g. road_link).
```
python3 script/integrate.py -c conf.json -T <theme> -t <table> -s 10
```

#### Integration after manual corrections

If a manal correction step has occurred before integration (for instance after the edge-matching process), it is usually done on a copy of the work table located in the "validation" schema of the database. Additional steps need to be followed to integrate the data at the end of the process:

##### 1) Extract the original data again
The original data needs to be extracted again with the same parameters as those used to extract it for the edge-matching process. This will fill the ids table (e.g. road_link_w_ids) which is used to detect deleted, created and modified objects.
```
python3 script/border_extract.py -c conf.json -T <theme> -t <table> -d <distance> <cc1> <cc2> 
```
##### 2) Empty the work table
This can be done with a SQL query: this action will empty the work table but keep the ids table correctly filled.
```
DELETE FROM <table>_w; 
```
##### 3) Fill the work table with the corrected data 
A SQL query is used to upload the data which was corrected manually into the work table, in order to launch the integration command in the next step.
```
INSERT INTO <table>_w SELECT * FROM validation.<cc1>_<cc2>_<table>_w_to_correct;
```
##### 4) Run the integration command line
This action will integrate the data from the work table into the original table.
```
python3 script/integrate.py -c conf.json -T <theme> -t <table>
```
