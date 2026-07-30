![under_construction](images/under_construction.png)

### OME2 database creation

Proceed the following ordered commands to create a complete OME2 database:


#### Initialisation de la structure
```
psql -h SMLPOPENMAPS2 -p 5432 -U postgres -d OME2 -f ./sql/db_init/HVLSP_0_GCMS_0_ADMIN.sql
psql -h SMLPOPENMAPS2 -p 5432 -U postgres -d OME2 -f ./sql/db_init/HVLSP_1_CREATE_SCHEMAS.sql
psql -h SMLPOPENMAPS2 -p 5432 -U postgres -d OME2 -f ./sql/db_init/ome2_reduce_precision_3d_trigger_function.sql
psql -h SMLPOPENMAPS2 -p 5432 -U postgres -d OME2 -f ./sql/db_init/ome2_reduce_precision_2d_trigger_function.sql
```


#### Création des tables

Création de toutes les tables pour l'ensemble des schémas:

```
python3 script/create_table.py -c conf.json -m mcd.json -T tn
python3 script/create_table.py -c conf.json -m mcd.json -T hy
python3 script/create_table.py -c conf.json -m mcd.json -T au
python3 script/create_table.py -c conf.json -m mcd.json -T ib
```


#### Historisation des tables
```
psql -h SMLPOPENMAPS2 -p 5432 -U postgres -d ome2_test_cd -f ./sql/db_init/HVLSP_2_GCMS_1_COMMON.sql
psql -h SMLPOPENMAPS2 -p 5432 -U postgres -d ome2_test_cd -f ./sql/db_init/HVLSP_3_GCMS_3_HISTORIQUE.sql
psql -h SMLPOPENMAPS2 -p 5432 -U postgres -d ome2_test_cd -f ./sql/db_init/ign_gcms_history_trigger_function.sql
psql -h SMLPOPENMAPS2 -p 5432 -U postgres -d ome2_test_cd -f ./sql/db_init/HVLSP_4_GCMS_4_OME2_ADD_HISTORY.sql
psql -h SMLPOPENMAPS2 -p 5432 -U postgres -d ome2_test_cd -c "ALTER SEQUENCE public.seqnumrec OWNER TO g_ome2_user;"
```