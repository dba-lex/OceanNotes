


- explain

```
EXPLAIN PLAN FOR
SELECT * FROM emp e, dept d
   WHERE e.deptno = d.deptno
   AND e.ename='benoit';

SET LINESIZE 130
SET PAGESIZE 0
SELECT * FROM table(DBMS_XPLAN.DISPLAY);
This query produces the following output:

Plan hash value: 3693697075
---------------------------------------------------------------------------
| Id  | Operation          | Name | Rows  | Bytes | Cost (%CPU)| Time     |
---------------------------------------------------------------------------
|   0 | SELECT STATEMENT   |      |     1 |    57 |     6  (34)| 00:00:01 |
|*  1 |  HASH JOIN         |      |     1 |    57 |     6  (34)| 00:00:01 |
|*  2 |   TABLE ACCESS FULL| EMP  |     1 |    37 |     3  (34)| 00:00:01 |
|   3 |   TABLE ACCESS FULL| DEPT |     4 |    80 |     3  (34)| 00:00:01 |
---------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------
  1 - access("E"."DEPTNO"="D"."DEPTNO")
  2 - filter("E"."ENAME"='benoit')

15 rows selected.
```

- display_cursor

```
SET PAGESIZE 0
SELECT * FROM table(DBMS_XPLAN.DISPLAY_CURSOR);

SELECT * FROM table(DBMS_XPLAN.DISPLAY_CURSOR('gwp663cqh5qbf',0));

alter session set statistics_level=all;
SELECT * FROM table (DBMS_XPLAN.DISPLAY_CURSOR('ggqns3c1jz86c', NULL, 'ALLSTATS LAST'));
```
