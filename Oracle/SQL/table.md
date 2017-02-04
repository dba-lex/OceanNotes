
- Displays partition information for the specified table, or all tables

```
SET LINESIZE 500
SET PAGESIZE 1000
SET FEEDBACK OFF
SET VERIFY OFF

SELECT a.table_name,
       a.partition_name,
       a.tablespace_name,
       a.initial_extent,
       a.next_extent,
       a.pct_increase,
       a.num_rows,
       a.avg_row_len
FROM   dba_tab_partitions a
WHERE  a.table_name  = Decode(Upper('&&1'),'ALL',a.table_name,Upper('&&1'))
AND    a.table_owner = Upper('&&2')
ORDER BY a.table_name, a.partition_name
/
```

- Displays the table statistics belonging to the specified schema

```
SET LINESIZE 300 VERIFY OFF

COLUMN owner FORMAT A20

SELECT owner,
       table_name,
       num_rows,
       blocks,
       empty_blocks,
       avg_space
       chain_cnt,
       avg_row_len,
       last_analyzed
FROM   dba_tables
WHERE  owner      = UPPER('&1')
AND    table_name = UPPER('&2');

SELECT index_name,
       blevel,
       leaf_blocks,
       distinct_keys,
       avg_leaf_blocks_per_key,
       avg_data_blocks_per_key,
       clustering_factor,
       num_rows,
       last_analyzed
FROM   dba_indexes
WHERE  table_owner = UPPER('&1')
AND    table_name  = UPPER('&2')
ORDER BY index_name;

COLUMN column_name FORMAT A30
COLUMN endpoint_actual_value FORMAT A30

SELECT column_id,
       column_name,
       num_distinct,
       avg_col_len,
       histogram,
       low_value,
       high_value
FROM   dba_tab_columns
WHERE  owner       = UPPER('&1')
AND    table_name  = UPPER('&2')
ORDER BY column_id;
```
