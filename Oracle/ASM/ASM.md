
- 查看ASM盘

```
set lines 400
col group_name for a20
col MOUNT_STATUS for a10
col mode_status for a10
col disk_name for a30
col path for a50
col failgroup for a30
select b.name GROUP_NAME,a.name DISK_NAME,a.failgroup,a.path,a.mount_status,a.mode_status
from v$asm_disk a,v$asm_diskgroup b where a.group_number = b.group_number order by 2;

select failgroup,name,path,mode_status,mount_status from v$asm_disk;
```

- 查看哪个数据库正在使用ASM

```
select group_number, db_name, status from v$asm_client;
```

- 查看ASM中每个盘的空间使用情况

```
set pages 100
set lines 400
col disk_group_name for a10
col disk_file_path for a60
col disk_file_name for a15
col disk_file_fail_group for a15
break on disk_group_name skip 2
compute sum of used_mb on disk_group_name


SELECT   NVL(a.name, '[CANDIDATE]')                   disk_group_name
       , b.path                                       disk_file_path
       , b.name                                       disk_file_name
       , b.failgroup                                  disk_file_fail_group
       , b.total_mb                                   total_mb
       , (b.total_mb - b.free_mb)                     used_mb
       , ROUND((1- (b.free_mb / b.total_mb))*100, 2)  pct_used
  FROM v$asm_diskgroup a RIGHT OUTER JOIN v$asm_disk b USING (group_number)
  where b.total_mb <> 0
 ORDER BY a.name
/
```

- 查看一个文件在ASM磁盘组中的AU分布

```
set lines 400
col path for a60
col failgroup for a15
SELECT  a.GROUP_KFFXP group_number,b.failgroup,b.path,count(a.AU_KFFXP) AU_count
 FROM x$kffxp a, v$asm_disk b, v$asm_alias c
WHERE a.number_kffxp = c.file_number
  and a.group_kffxp = c.group_number
  AND a.GROUP_KFFXP = b.group_number
  AND a.disk_kffxp = b.disk_number
  AND c.name = upper('&filename')
group by a.GROUP_KFFXP,b.failgroup,b.path
;
```


- Partners

```
SELECT count(disk_number)
FROM v$asm_disk
WHERE group_number = 2;


SELECT disk "Disk", count(number_kfdpartner) "Number of partners"
FROM x$kfdpartner
WHERE grp=2
GROUP BY disk
ORDER BY 1;

set pages 1000
break on Group# on Disk#
SELECT d.group_number "Group#", d.disk_number "Disk#", p.number_kfdpartner "Partner disk#"
FROM x$kfdpartner p, v$asm_disk d
WHERE p.disk=d.disk_number and p.grp=d.group_number
ORDER BY 1, 2, 3;
```

- ASM元信息文件所在的AU

```
select NUMBER_KFFXP "ASM file number",
        DECODE (NUMBER_KFFXP,
            1, 'File directory',
            2, 'Disk directory',
            3, 'Active change directory',
            4, 'Continuing operations directory',
            5, 'Template directory',
            6, 'Alias directory',
            7, 'ADVM file directory',
            8, 'Disk free space directory',
            9, 'Attributes directory',
            10, 'ASM User directory',
            11, 'ASM user group directory',
            12, 'Staleness directory',
            253, 'spfile for ASM instance',
            254, 'Stale bit map space registry ',
            255, 'Oracle Cluster Repository registry') "ASM metadata file name",
count(AU_KFFXP) "Allocation units"
from X$KFFXP
where GROUP_KFFXP=1
    and NUMBER_KFFXP<256
group by NUMBER_KFFXP
order by 1;
```

- 查询一个文件的extent和AU分布

```
select XNUM_KFFXP "Virtual extent",
        PXN_KFFXP "Physical extent",
        LXN_KFFXP "Extent copy",
        DISK_KFFXP "Disk",
        AU_KFFXP "Allocation unit"
from X$KFFXP
where GROUP_KFFXP=1
    and NUMBER_KFFXP=256
    and XNUM_KFFXP<>2147483648
order by 1,2;
```

- 磁盘组数据平衡的状态

```
column "Diskgroup" format A30
column "Imbalance" format 99.99 Heading "Percent|Imbalance"
column "Variance" format 99.99 Heading "Percent|Disk Size|Variance"
column "MinFree" format 99.99 Heading "Minimum|Percent|Free"
column "DiskCnt" format 9999 Heading "Disk|Count"
column "Type" format A10 Heading "Diskgroup|Redundancy"
SELECT g.name "Diskgroup",
100*(max((d.total_mb-d.free_mb)/d.total_mb)-min((d.total_mb-d.free_mb)/d.total_mb))/max((d.total_mb-d.free_mb)/d.total_mb) "Imbalance",
100*(max(d.total_mb)-min(d.total_mb))/max(d.total_mb) "Variance",
100*(min(d.free_mb/d.total_mb)) "MinFree",
count(*) "DiskCnt",
g.type "Type"
FROM
    v$asm_disk d,
    v$asm_diskgroup g
WHERE
    d.group_number = g.group_number
    and d.group_number <> 0
    and d.state = 'NORMAL'
    and d.mount_status = 'CACHED'
GROUP BY
    g.name,
    g.type;
```

- 磁盘组中文件号、文件名、文件类型

```
col name for a60
col type for a25
SELECT f.group_number, f.file_number, a.name, f.type
FROM v$asm_file f, v$asm_alias a
WHERE f.group_number=a.group_number and f.file_number=a.file_number
ORDER BY 1, 2;
```
