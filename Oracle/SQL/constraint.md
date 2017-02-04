
- DBA_constraints.CONSTRAINT_TYPE

```
C:  Check ,On Table
O:  Read Only, On View
P:  Primary Key
R:  FK(Foreign Key, Referential Integrity)
U:  Unique Key
V:  Check Option, On View
```

- 查询某表上的外键

```
col column_name for a30
col owner for a20
set lines 400
SELECT cols.table_name, cols.column_name, cols.position, cons.status, cons.owner
FROM all_constraints cons, all_cons_columns cols
WHERE cols.table_name = 'SALES'
AND cons.constraint_type = 'R'
AND cons.constraint_name = cols.constraint_name
AND cons.owner = cols.owner
ORDER BY cols.table_name, cols.position;
```

- Displays a list of non-indexes FKs

```
SET SERVEROUTPUT ON
SET PAGESIZE 1000
SET LINESIZE 255
SET FEEDBACK OFF

SELECT t.table_name,
       c.constraint_name,
       c.table_name table2,
       acc.column_name
FROM   all_constraints t,
       all_constraints c,
       all_cons_columns acc
WHERE  c.r_constraint_name = t.constraint_name
AND    c.table_name        = acc.table_name
AND    c.constraint_name   = acc.constraint_name
AND    NOT EXISTS (SELECT '1'
                   FROM  all_ind_columns aid
                   WHERE aid.table_name  = acc.table_name
                   AND   aid.column_name = acc.column_name)
ORDER BY c.table_name;


PROMPT
SET FEEDBACK ON
SET PAGESIZE 18
```

- 查询某表上的主键

```
col column_name for a30
col owner for a20
set lines 400
SELECT cols.table_name, cols.column_name, cols.position, cons.status, cons.owner
FROM all_constraints cons, all_cons_columns cols
WHERE cols.table_name = 'PRODUCTS'
AND cons.constraint_type = 'P'
AND cons.constraint_name = cols.constraint_name
AND cons.owner = cols.owner
ORDER BY cols.table_name, cols.position;
```
