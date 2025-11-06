# Define international boundaries for OME2

For the OME2 project, in order to harmonise data across countries, it is necessary to use common international boundaries. However, neighbouring countries often provide different geometries for their international boundaries, even when these are officially agreed.

An FME workbench was therefore put into place to calculate common “technical” boundaries, which are considered as a reference for the project: boundary_unification_process

## 1. How to use the workbench
This workbench is divided into 8 steps. Each step produces output tables which are then used as input in the following step.
The steps have to be launched one by one, in order to allow for some manual work in between.
Before launching a step, make sure to:
- Activate the tables which need to be read.
- Deactivate any link from these tables to transformers from other steps (cf. steps 2.1 and 4.1).

## 2. Workbench description
Example: BE#NL
Pre-requisite: both countries’ administrative units themes need to have been included in the OME2 database. There must be a country polygon for each country in the au.administrative_unit_area_1 table (or possibly several polygons in case of islands etc.).
The au.administrative_unit_area_1 from the OME2 PostgreSQL database will be used as source data for the process.

### Step 0 (FME): Configure workbench
The workbench parameters first need to be configured directly in the workbench:

 <img width="388" height="243" alt="image" src="https://github.com/user-attachments/assets/8cece253-f3d9-4abe-b968-6d0d59865061" />
 
There are three of them:
- Country code 1: country code of the first country to be processed in alphabetical order (referred to as “cc1” in the document).
- Country code 2: country code of the second country to be processed in alphabetical order (referred to as “cc2” in the document).
- OME2 database: name of the database connection configured in FME which gives access to the PostgreSQL OME2 database. If the connection does not exist in FME, it needs to be created first.

### Step 1.1 (FME): Extract outer international boundaries from administrative units.
##### Input:
- au.administrative_unit_area_1
##### TO-DO in FME:
- Enable only the au.administrative_unit_area_1 feature type (Step 1.1 frame).
- Run workbench.
##### Steps description:
1.	For each country, dissolve administrative units into single part polygons.
2.	Extract edges from the dissolved polygons  these are the outer boundaries, which then need to be cleaned.
3.	Slightly simplify these outer boundaries to avoid having too many vertices (Douglas-Peucker algorithm with a 10-cm tolerance).
4.	Gather edges from the two countries into a single table and planarize the data.
5.	Merge the lines to remove pseudo-nodes: the remaining lines are split only where at least 3 objects intersect.

<img width="945" height="162" alt="image" src="https://github.com/user-attachments/assets/72fe3e53-90f8-4e4b-827a-b39cc097e3fd" />

##### Output:
- public._intbnd1_ lines (backup copy)
- public._intbnd1_lines_corrected (to be corrected in the next step).

<img width="522" height="447" alt="image" src="https://github.com/user-attachments/assets/6794a103-e4e1-4823-a1af-96f460fad235" />

Since we are working on the BE#NL international boundary, all lines which do not belong to this boundary need to be deleted manually in the next step.

### Step 1.2 (manual): Clean the outer edges to keep only BE#NL data 

##### Input:
- public._intbnd1_lines_corrected (before correction)

##### Steps to be performed (in QGIS for example):
1.	Manually delete the lines which do not correspond to the BE#NL boundary (coastlines, boundaries with other countries, Belgian enclaves in Germany). To do that in QGIS you can:
- Activate snapping
- Use the “separate entities” tool to split lines where the boundary to be processed stops  
2.	If necessary, snap the endpoints of the two lines representing the BE and NL version of the main international boundary (for the test, it was necessary on the junction with Germany). If the neighbouring boundaries are already edge-matched, there should be an international_boundary_node object which should be used as reference (or updated if necessary). Otherwise, the endpoints should be snapped approximately in the middle of the space separating them (the distance is usually very small so the precise location is not very important).
3.	If there is no international_boundary_node object at the location of the new intersection, create one and fill its country_code attribute with the country codes of all intersecting countries, in alphabetical order and separated by a hash sign (#).

| Initial state    | After correction |
|-------------------|------------------|
|<img width="441" height="281" alt="image" src="https://github.com/user-attachments/assets/a49fd4be-8c9b-473a-aa59-0d120636c3b7" /> | <img width="385" height="288" alt="image" src="https://github.com/user-attachments/assets/ecdb6036-3da3-44f7-b4b2-79814392f312" /> |

##### Output:
- public._intbnd1_lines_corrected (corrected version)
- ib._international_boundary_node

<img width="893" height="310" alt="image" src="https://github.com/user-attachments/assets/37709ce7-7386-43bf-ab1c-71399a37ff50" />

### Step 2.1 (FME): Create polygons wherever the two versions of the boundary differ 
##### Input:
•	public._intbnd1_lines_corrected (corrected version)
##### TO-DO in FME:
- Enable only the _intbnd1_lines_corrected feature type (it is located on the left-hand side of the workbench, near step 3). 
- Enable the link between _intbnd1_lines_corrected and the AttributeRemover tranformer in section Step 2.1.
- Then run the workbench.
- Disable the link between _intbnd1_lines_corrected and the AttributeRemover tranformer in section Step 2.1.
##### Steps description:
Generate polygons from the input table -> all in-dispute areas between the two boundaries are transformed into polygons. However, this also creates polygons inside enclaves, which need to be deleted before the next step because they are not needed to generate the common boundary.
<img width="945" height="175" alt="image" src="https://github.com/user-attachments/assets/564e5ee9-6ce8-408d-ac61-92b467cfb949" />

##### Output:
- public._intbnd2_polygons (backup copy)
- public._intbnd2_polygons (to be corrected in the next step).
- public._intbnd2_agreed_lines

<img width="878" height="436" alt="image" src="https://github.com/user-attachments/assets/f5fc786f-6af5-4a54-9039-f2f341bd824c" />

### Step 2.2 (manual): Delete polygons corresponding to enclaves 
##### Input:
- public._intbnd2_polygons_corrected (before correction).
##### Steps to be performed:
Delete all enclaves i.e. all the polygons inside which we shouldn’t calculate a new boundary.
##### Output:
- public._intbnd2_polygons_corrected (corrected version).

<img width="698" height="445" alt="image" src="https://github.com/user-attachments/assets/ae00d5c8-18b3-45fb-a029-4effb09ff69e" />

### Step 3 (FME): Generate skeletons (~centerlines) from the boundary polygons
##### Input:
- public._intbnd2_polygons_corrected (corrected version).
- public._intbnd1_lines_corrected (corrected version).
##### TO-DO in FME:
- Enable only the _intbnd2_polygons_corrected feature type.
- Enable _intbnd1_lines_corrected (near Step 3).
- Enable the link between _intbnd1_lines_corrected and the LineOnAreaOverlayer transformer in section Step 3.
- Make sure link between _intbnd1_lines_corrected and the AttributeRemover transformer (in section Step 2.1) is disabled.
- Run workbench.
- Disable the link between _intbnd1_lines_corrected and the LineOnAreaOverlayer transformer in section Step 3.

##### Steps description:
1.	Retrieve lines which did not generate polygons: this situation occurs when the two original country polygons actually share a boundary. To do that, the workbench looks for lines from _intbnd1_lines_corrected which do not overlap the polygons created above (_intbnd2_polugons_corrected). Such segments are stored in _intbnd3_agreed_lines and will be added to the final boundary table at the end of the process. 
2.	Generate the skeletons of all polygons from the corrected table.
3.	Use Intersector to planarize the resulting lines.
4.	Manage attributes to keep only the information needed for OME2.
<img width="945" height="190" alt="image" src="https://github.com/user-attachments/assets/a2abf218-1f0c-4664-ad9e-6328f6df24b2" />

##### Output:
- public._intbnd3_skeleton (backup copy).

<img width="678" height="372" alt="image" src="https://github.com/user-attachments/assets/f7af85f3-96f8-471f-a2a1-13820330bbec" />

From these skeletons, we want to extract the main central line i.e. eliminate all small objects growing out of the central line.

### Step 4.1 (FME): Identify irrelevant lines to be deleted
The lines to be kept answer to the following characteristics:
-	Either they do not intersect _intbnd1_lines_corrected at all (represented as blue dashed lines in the following pictures),
-	OR they intersect at least three objects from _intbnd1_lines_corrected,
-	OR they are located at the very end of the main international boundary line and therefore intersect an object from ib.international_boundary_node.

| Skeleton line intersecting no boundary line | Skeleton line intersecting 4 boundary lines | Skeleton line intersecting 3 boundary lines |
|---------------------------------------------|---------------------------------------------|---------------------------------------------|
|<img width="290" height="198" alt="image" src="https://github.com/user-attachments/assets/1b275d40-eefc-46b8-a590-966fc30472ae" /> | <img width="449" height="282" alt="image" src="https://github.com/user-attachments/assets/3e19f9db-1757-437d-83eb-188e10d45b0c" /> | <img width="452" height="264" alt="image" src="https://github.com/user-attachments/assets/20adf183-e813-4e8e-b204-1ee5b676cb6f" /> |

##### Input:
- public._intbnd3_skeleton.
- public._intbnd1_lines_corrected (corrected version).
- ib.international_boundary_node
##### TO-DO in FME:
- Enable the 3 feature types mentioned above (use “Enable only” on the first one, then enable the other two).
- Make sure no other feature types are activated.
- Make sure the links between _intbnd1_lines_corrected and the AttributeRemover (Step 2.1) and LineOnAreaOverlayer transformers (Step 3) are disabled.
- Run workbench.

##### Steps description:
1.	Initial selection:
 a.	Select objects from _intbnd3_skeleton which intersect ib_boundary_nodes -> they are kept in the initial selection.
 b.	Remove the objects from _INTBND3_skeleton which intersect only one or two objects from _INTBND1_lines_corrected.
 c.	Dissolve all the selected objects from _INTBND3_skeleton so that they are split only at the intersection of three or more objects.
 d.	Manage UIDs.
<img width="945" height="403" alt="image" src="https://github.com/user-attachments/assets/757beeea-107b-4cf7-9b48-14a42d60cae0" />

2.	Clean the resulting objects: at this stage, there are still some small dangling segments sprouting from the main line in the table. This step aims at deleting as many of these dangles as possible.
 a.	Generate nodes from the objects coming out of step1.
 b.	Identify dangling nodes by selecting nodes, i.e. nodes linked only to 1 skeleton object from step 1.
 c.	Among these nodes, some are actually connected to loops and are not really dangling nodes: remove them from the selected nodes.
 d.	Keep only the objects from step 1 which do not intersect the selected dangling nodes -> these objects are recorded in _INTBND4_corrected (to be corrected in the next step).
<img width="944" height="241" alt="image" src="https://github.com/user-attachments/assets/1dc52f9d-2f3b-4ed4-980e-d240edef5130" />

 e.	Generate nodes from these selected skeleton objects and select those which are connected to at least 3 objects -> record them in _INTBND4_suspicious_nodes.
 
<img width="552" height="192" alt="image" src="https://github.com/user-attachments/assets/57d26dfb-64b4-4f4d-beac-4831ef007020" />

##### Output:
- public._intbnd4_corrected (before correction).
- public._intbnd4_suspicious_nodes (not created if empty).

### Step 4.2 (manual): Delete irrelevant objects
##### Input:
- public._intbnd4_corrected (before correction).
- public._intbnd4_suspicious_nodes.
Steps to be performed:
1.	In QGIS, open both tables and go through the suspicious nodes. Each node should correspond to the location of an object sprouting from the main line, to be deleted.
2.	Manually delete the irrelevant objects.

| Case 1: small segment connecting two sides of a polygon | Case 2: intermediate dangle which gave birth to several other dangles in the initial skeleton table | 
|---------------------------------------------|---------------------------------------------|
|<img width="463" height="260" alt="image" src="https://github.com/user-attachments/assets/f3954ea6-f7fe-4761-b1a9-18dbba822d1e" /> | <img width="317" height="266" alt="image" src="https://github.com/user-attachments/assets/27bf7ccf-f9d4-405e-9137-23d5d714f3a4" /> | 

##### Output:
- public._intbnd4_corrected (corrected version).

### Step 4.3 (FME): Finalization
##### Input:
- public._intbnd4_corrected (corrected version).
- public._intbnd2_agreed_lines.
##### TO-DO in FME:
- In the Step 4.3 frame, enable public._intbnd4_corrected feature type and public._intbnd2_agreed_lines (make sure no other feature type is enabled).
- Run workbench.
##### Steps description:
1.	Combine _intbnd4_corrected and _intbnd3_agreed_lines.
2.	Dissolve all lines.
3.	Slightly generalize them in order to avoid having too many vertices (Douglas-Peucker algorithm with a 10-cm tolerance).
4.	Add UUIDs and clean attributes. Note that legal_status will be set to “not_agreed” and technical_status will be set to “edge_matched”.
5.	Generate suspicious nodes to control the data: this table should be empty.

<img width="945" height="191" alt="image" src="https://github.com/user-attachments/assets/6909aeac-9881-4693-ab2a-0d35d42e311c" />

> Note: it was originally planned to distinguish agreed portions of the boundary (coming from _intbnd3_agreed_lines) from calculated lines in the final table. However, agreed portions could be very small and create errors in the edge-matching process. Two options were considered:
> -	Keeping agreed lines longer than a given length threshold.
> -	Merging all lines.
> Since there was no clear requirement to make this distinction, the second option was kept.

##### Output:
- public._intbnd5_boundary_final_xx_yy (where xx and yy are the country codes of the countries to process)
- public._intbnd5_suspicious_nodes_verification_xx_yy (not created if empty)  this table should not be created. If it is, go through the new suspicious nodes and correct _intbnd4_corrected on these locations. Then launch step 4.3 again.

### Step 4.4 (PgAdmin): Integration into ib.international_boundary_line
**WARNING: to be performed by one of the database’s administrators**
````
UPDATE ib.international_boundary_line SET gcms_detruit = true WHERE country = 'country1#country2';

INSERT INTO ib.international_boundary_line (objectid, begin_lifespan_version, country, legal_status, technical_status, geom, boundary_type, boundary_source)
SELECT objectid, begin_lifespan_version, country, legal_status, technical_status, geom, boundary_type, xy_source
FROM _intbnd5_boundary_final_xx_yy;
````











---------
> TO-DO: move to boundary_unification doc
> - When an isolated country is integrated, only the first step of the workbench needs to be used. This creates the boundary lines (including coastlines) all around the country.
> - Then the lines need to be split manually where there is a change in country code or boundary type (international boundary vs coastline).
> - Create point in ib.international_boundary_node when there is a change in the boundary line (type or country code).
> - Use EBM as reference.
