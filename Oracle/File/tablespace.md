


- 获取表空间DDL

```
SET LONG 20000 LONGCHUNKSIZE 20000 PAGESIZE 0 LINESIZE 1000 FEEDBACK OFF VERIFY OFF TRIMSPOOL ON

BEGIN
   DBMS_METADATA.set_transform_param (DBMS_METADATA.session_transform, 'SQLTERMINATOR', true);
   DBMS_METADATA.set_transform_param (DBMS_METADATA.session_transform, 'PRETTY', true);
END;
/

SELECT DBMS_METADATA.get_ddl ('TABLESPACE', tablespace_name)
FROM   dba_tablespaces
WHERE  tablespace_name = DECODE(UPPER('&1'), 'ALL', tablespace_name, UPPER('&1'));

SET PAGESIZE 14 LINESIZE 100 FEEDBACK ON VERIFY ON
```



alter tablespace users nologging;


- datafile resize

```
-- -----------------------------------------------------------------------------------
-- File Name    : http://www.oracle-base.com/dba/monitoring/min_datafile_size.sql
-- Author       : DR Timothy S Hall
-- Description  : Displays smallest size the datafiles can shrink to without a reorg.
-- Requirements : Access to the V$ and DBA views.
-- Call Syntax  : @min_datafile_size
-- Last Modified: 07/09/2007
-- -----------------------------------------------------------------------------------

COLUMN block_size NEW_VALUE v_block_size

SELECT TO_NUMBER(value) AS block_size
FROM   v$parameter
WHERE  name = 'db_block_size';

COLUMN tablespace_name FORMAT A20
COLUMN file_name FORMAT A50
COLUMN current_bytes FORMAT 999999999999999
COLUMN shrink_by_bytes FORMAT 999999999999999
COLUMN resize_to_bytes FORMAT 999999999999999
SET VERIFY OFF
SET LINESIZE 200

SELECT a.tablespace_name,
       a.file_name,
       a.bytes AS current_bytes,
       a.bytes - b.resize_to AS shrink_by_bytes,
       b.resize_to AS resize_to_bytes
FROM   dba_data_files a,
       (SELECT file_id, MAX((block_id+blocks-1)*&v_block_size) AS resize_to
        FROM   dba_extents
        GROUP by file_id) b
WHERE  a.file_id = b.file_id
ORDER BY a.tablespace_name, a.file_name;
```
