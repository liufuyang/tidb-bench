TPC-H Benchmark, specific for MySQL/TiDB
## Introduction
file structure
  - alltable.load
	mysql commands to load all data
  - alltable.count
	mysql commands to select count(*) all tables
  - dss.sql
 	modified schema definition file
  - get-all-query-time.pl
 	perl script to get runtime of each query
  - get-query-time.pl
 	perl script to get runtime of a query
  - queries
	22 TPC-H queries
  - tablescanquery.pl
	perl script to generate scan query for all tables

Some changes have been made to official files provided by TPC-H, to make
them work with TiDB.

## How to use 

Use mysql client login to `TiDB` and create a databse call `tpch`.
`make tbl` will make `dbgen` and generate `*.tbl` files.

`make load` will create database and tables in TiDB and load all `*.tbl` into TiDB.

After this, please be patient. If you really want to know any progress that `TiDB` is
making right now, you can open another terminal and use query defined in `alltable.count`
to know how many rows in the table.

If you are using macOs, please replace `#include<malloc.h>` in `varsub.c` and `bm_utils.c` to `#include<stdlib.c>`.

## Query examples

Query 1
```sql
SELECT l_returnflag, 
       l_linestatus, 
       SUM(l_quantity)                                           AS sum_qty, 
       SUM(l_extendedprice)                                      AS 
       sum_base_price, 
       SUM(l_extendedprice * ( 1 - l_discount ))                 AS 
       sum_disc_price, 
       SUM(l_extendedprice * ( 1 - l_discount ) * ( 1 + l_tax )) AS sum_charge, 
       Avg(l_quantity)                                           AS avg_qty, 
       Avg(l_extendedprice)                                      AS avg_price, 
       Avg(l_discount)                                           AS avg_disc, 
       Count(*)                                                  AS count_order 
FROM   lineitem 
WHERE  l_shipdate <= DATE '1998-12-01' - interval '90' day 
GROUP  BY l_returnflag, 
          l_linestatus 
ORDER  BY l_returnflag, 
          l_linestatus; 
```

Query 2
```sql
SELECT s_acctbal, 
       s_name, 
       n_name, 
       p_partkey, 
       p_mfgr, 
       s_address, 
       s_phone, 
       s_comment 
FROM   part, 
       supplier, 
       partsupp, 
       nation, 
       region 
WHERE  p_partkey = ps_partkey 
       AND s_suppkey = ps_suppkey 
       AND p_size = 15 
       AND p_type LIKE '%BRASS' 
       AND s_nationkey = n_nationkey 
       AND n_regionkey = r_regionkey 
       AND r_name = 'EUROPE' 
       AND ps_supplycost = (SELECT Min(ps_supplycost) 
                            FROM   partsupp, 
                                   supplier, 
                                   nation, 
                                   region 
                            WHERE  p_partkey = ps_partkey 
                                   AND s_suppkey = ps_suppkey 
                                   AND s_nationkey = n_nationkey 
                                   AND n_regionkey = r_regionkey 
                                   AND r_name = 'EUROPE') 
ORDER  BY s_acctbal DESC, 
          n_name, 
          s_name, 
          p_partkey 
LIMIT  100; 
```