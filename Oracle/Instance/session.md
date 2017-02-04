
- Displays memory allocations for the current database sessions

```
SET LINESIZE 200

COLUMN username FORMAT A20
COLUMN module FORMAT A20

SELECT NVL(a.username,'(oracle)') AS username,
       a.module,
       a.program,
       Trunc(b.value/1024) AS memory_kb
FROM   v$session a,
       v$sesstat b,
       v$statname c
WHERE  a.sid = b.sid
AND    b.statistic# = c.statistic#
AND    c.name = 'session pga memory'
AND    a.program IS NOT NULL
ORDER BY b.value DESC;
```

- Displays the SQL statement held for a specific SID

```
SET LINESIZE 500
SET PAGESIZE 1000
SET VERIFY OFF

SELECT a.sql_text
FROM   v$sqltext a,
       v$session b
WHERE  a.address = b.sql_address
AND    a.hash_value = b.sql_hash_value
AND    b.sid = &1
ORDER BY a.piece;

PROMPT
SET PAGESIZE 14
```

- Displays the SQL statement held for a specific SID

```
SET LINESIZE 500
SET PAGESIZE 1000
SET VERIFY OFF

SELECT oc.sql_text, cursor_type
FROM   v$open_cursor oc
WHERE  oc.sid = &1
ORDER BY cursor_type;
```
