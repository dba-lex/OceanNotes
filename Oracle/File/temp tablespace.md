
临时表空间用来管理数据库排序操作以及用于存储临时表、中间排序结果等临时对象,当ORACLE里需要用到SORT的时候，并且当PGA中sort_area_size大小不够时，将会把数据放入临时表空间里进行排序。像数据库中一些操作： CREATE INDEX、 ANALYZE、SELECT DISTINCT、ORDER BY、GROUP BY、 UNION ALL、 INTERSECT、MINUS、SORT-MERGE JOINS、HASH JOIN等都可能会用到临时表空间。当操作完成后，系统会自动清理临时表空间中的临时对象，自动释放临时段。这里的释放只是标记为空闲、可以重用，其实实质占用的磁盘空间并没有真正释放。这也是临时表空间有时会不断增大的原因。

临时表空间存储大规模排序操作(小规模排序操作会直接在RAM里完成，大规模排序才需要磁盘排序Disk Sort)和散列操作的中间结果.它跟永久表空间不同的地方在于它由临时数据文件（temporary files）组成的,而不是永久数据文件（datafiles）。临时表空间不会存储永久类型的对象，所以它不会也不需要备份。另外，对临时数据文件的操作不产生redo日志，不过会生成undo日志。

创建临时表空间或临时表空间添加临时数据文件时，即使临时数据文件很大，添加过程也相当快。这是因为ORACLE的临时数据文件是一类特殊的数据文件:稀疏文件（Sparse File）,当临时表空间文件创建时，它只会写入文件头部和最后块信息(only writes to the header and last block of the file)。它的空间是延后分配的.这就是你创建临时表空间或给临时表空间添加数据文件飞快的原因。

另外，临时表空间是NOLOGGING模式以及它不保存永久类型对象，因此即使数据库损毁，做Recovery也不需要恢复Temporary Tablespace。


- 查看默认temp表空间

```
SELECT PROPERTY_NAME, PROPERTY_VALUE
FROM DATABASE_PROPERTIES
WHERE PROPERTY_NAME='DEFAULT_TEMP_TABLESPACE';
```

- 查看某个用户的temp表空间

```
SELECT USERNAME, TEMPORARY_TABLESPACE FROM DBA_USERS;
```

- 查看temp文件

```
SET LINESIZE 1200
COL TABLESPACE_NAME FOR A30
COL FILE_NAME FOR A50
SELECT TABLESPACE_NAME                 AS TABLESPACE_NAME
        ,FILE_NAME                     AS FILE_NAME
        ,BLOCKS                        AS BLOCKS
        ,STATUS                        AS STATUS
        ,AUTOEXTENSIBLE                AS AUTOEXTENSIBLE
        ,BYTES/1024/1024/1024          AS "FILE_SIZE(G)"
        ,DECODE(MAXBYTES, 0, BYTES/1024/1024/1024,
                          MAXBYTES/1024/1024/1024)
                                       AS "MAX_SIZE(G)"
        ,INCREMENT_BY                  AS "INCREMENT_BY"
        ,USER_BYTES/1024/1024/1024     AS "USEFUL_SIZE"
FROM DBA_TEMP_FILES;


SET LINESIZE 1200
COL NAME FOR A60
SELECT FILE#                        AS FILE_NUMBER
    ,NAME                           AS NAME
    ,CREATION_TIME                  AS CREATION_TIME
    ,BLOCK_SIZE                     AS BLOCK_SIZE
    ,BYTES/1024/1024/1024           AS "FILE_SIZE(G)"
    ,CREATE_BYTES/1024/1024/1024    AS "INIT_SIZE(G)"
    ,STATUS                         AS STATUS
    ,ENABLED                        AS ENABLED
FROM V$TEMPFILE;
```

- tempfile rename

```
ALTER DATABASE TEMPFILE '/u01/app/oracle/oradata/GSP/temp4.dbf' OFFLINE;
mv /u01/app/oracle/oradata/GSP/temp4.dbf /u01/app/oracle/oradata/GSP/temp04.dbf
ALTER DATABASE RENAME FILE '/u01/app/oracle/oradata/GSP/temp4.dbf' TO '/u01/app/oracle/oradata/GSP/temp04.dbf';
ALTER DATABASE TEMPFILE '/u01/app/oracle/oradata/GSP/temp04.dbf' ONLINE;
```

- 临进表空间组:

```
临进表空间组是ORACLE 10g引入的一个新特性，它是一个逻辑概念，不需要显示的创建和删除。只要把一个临时表空间分配到一个组中，临时表空间组就自动创建，所有的临时表空间从临时表空间组中移除就自动删除。

一个临时表空间组必须由至少一个临时表空间组成，并且无明确的最大数量限制.
```

- 监控临时表空间使用

```
SELECT TU.TABLESPACE_NAME                                    AS "TABLESPACE_NAME",
       TT.TOTAL - TU.USED                                    AS "FREE(G)",
       TT.TOTAL                                              AS "TOTAL(G)",
       ROUND(NVL(TU.USED, 0) / TT.TOTAL * 100, 3)            AS "USED(%)",
       ROUND(NVL(TT.TOTAL - TU.USED, 0) * 100 / TT.TOTAL, 3) AS "FREE(%)"
FROM (SELECT TABLESPACE_NAME,
              SUM(BYTES_USED) / 1024 / 1024 / 1024 USED
       FROM GV_$TEMP_SPACE_HEADER
       GROUP BY TABLESPACE_NAME) TU ,
     (SELECT TABLESPACE_NAME,
              SUM(BYTES) / 1024 / 1024 / 1024 AS TOTAL
       FROM DBA_TEMP_FILES
       GROUP BY TABLESPACE_NAME) TT
WHERE TU.TABLESPACE_NAME = TT.TABLESPACE_NAME;


COL TEMP_FILE FOR A60;
SELECT ROUND((F.BYTES_FREE  + F.BYTES_USED)/1024/1024/1024, 2)                         AS "TOTAL(GB)",
       ROUND(((F.BYTES_FREE  + F.BYTES_USED) - NVL(P.BYTES_USED, 0))/1024/1024/1024,2) AS "FREE(GB)",
       D.FILE_NAME                                                                     AS "TEMP_FILE",
       ROUND(NVL(P.BYTES_USED, 0)/1024/1024/1024, 2)                                   AS "USED(GB)" ,
       ROUND((F.BYTES_USED + F.BYTES_FREE)/1024/1024/1024, 2)                          AS "TOTAL(GB)",
       ROUND(((F.BYTES_USED + F.BYTES_FREE) - NVL(P.BYTES_USED, 0))/1024/1024/1024, 2) AS "FREE(GB)" ,
       ROUND(NVL(P.BYTES_USED, 0)/1024/1024/1024, 2)                                   AS "USED(GB)"
FROM SYS.V_$TEMP_SPACE_HEADER F ,DBA_TEMP_FILES D ,SYS.V_$TEMP_EXTENT_POOL P
WHERE F.TABLESPACE_NAME(+) = D.TABLESPACE_NAME
  AND F.FILE_ID(+) = D.FILE_ID
  AND P.FILE_ID(+) = D.FILE_ID;
```
