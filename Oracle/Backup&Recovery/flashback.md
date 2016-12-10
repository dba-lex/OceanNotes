
- 设置FRA大小及路径：

```
ALTER SYSTEM SET db_recovery_file_dest_size=100G SCOPE=SPFILE;
ALTER SYSTEM SET DB_RECOVERY_FILE_DEST='+FLASHDG' SCOPE=SPFILE;
show parameter db_recovery_file_dest
SELECT * FROM V$RECOVERY_FILE_DEST;
```


- FRA空间使用情况

```
set lines 100
col name format a60
 
-- FRA Occupants
SELECT * FROM V$RECOVERY_AREA_USAGE;
SELECT * FROM V$FLASH_RECOVERY_AREA_USAGE;

col name for a32
col size_m for 999,999,999
col reclaimable_m for 999,999,999
col used_m for 999,999,999
col pct_used for 999
SELECT name,
    ceil( space_limit / 1024 / 1024) SIZE_M,
    ceil( space_used  / 1024 / 1024) USED_M,
    ceil( space_reclaimable  / 1024 / 1024) RECLAIMABLE_M,
    decode( nvl( space_used, 0),0, 0,
    ceil(((space_used - space_reclaimable) / space_limit) * 100) ) PCT_USED
FROM v$recovery_file_dest
ORDER BY name
/
```
