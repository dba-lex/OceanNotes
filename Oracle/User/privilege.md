
- Displays object privileges on a specified object

```
SET LINESIZE 200 VERIFY OFF

SELECT owner,
       table_name AS object_name,
       grantor,
       grantee,
       privilege,
       grantable,
       hierarchy
FROM   dba_tab_privs
WHERE  owner      = UPPER('&1')
AND    table_name = UPPER('&2')
ORDER BY 1,2,3,4;

SET VERIFY ON
```
