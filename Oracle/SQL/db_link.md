
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

- 创建dblink

```
create database link source connect to system IDENTIFIED BY oracle
using 'dbalex';

create database link source
  connect to system IDENTIFIED BY oracle
  using '(DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 10.10.90.200)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl)
    )
  )';
```
