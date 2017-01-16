
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
