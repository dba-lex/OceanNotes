
- Displays information on all database links

```
SET LINESIZE 150

COLUMN owner FORMAT A30
COLUMN db_link FORMAT A30
COLUMN username FORMAT A30
COLUMN host FORMAT A30

SELECT owner,
       db_link,
       username,
       host
FROM   dba_db_links
ORDER BY owner, db_link;
```
