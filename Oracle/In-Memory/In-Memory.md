
- 会话级别关闭In-Memory查询

```
alter session set inmemory_query=disable;
```

- 设置In-Memory内存区大小

```
alter system set sga_target=320G scope=spfile;
alter system set inmemory_size=300G scope=spfile;
```

- 查看In-Memory内存区内存分配

```
select * from v$inmemory_area;
```

- 查看In-Memory内存区中表的加载情况

```
select SEGMENT_NAME,INMEMORY_SIZE,BYTES,POPULATE_STATUS from v$im_segments;
```

- 查看会话中In-Memory的使用统计信息

```
select name,value from v$mystat, v$statname
where v$mystat.statistic# = v$statname.statistic#
  and v$statname.name in
  ('CPU used by this session',
   'IM scan rows',
   'IM scan rows valid',
   'IM scan CUs no memcompress',
   'IM scan CUs pruned'
    );
```

- 设置表test加载到In-Memory内存区

```
alter table test inmemory NO MEMCOMPRESS;
ALTER TABLE test inmemory MEMCOMPRESS FOR CAPACITY LOW;
ALTER TABLE test inmemory PRIORITY HIGH;
ALTER TABLE test inmemory MEMCOMPRESS FOR DML;
ALTER TABLE test inmemory MEMCOMPRESS FOR QUERY;
ALTER TABLE test inmemory MEMCOMPRESS FOR CAPACITY;
```

- 关闭表的In-Memory列式存储属性

```
ALTER TABLE test NO INMEMORY;
```

- 计算In-Memory内存区中表的压缩比

```
COL OWNER FORMAT a5
COL SEGMENT_NAME FORMAT a5
SET PAGESIZE 50000

SELECT v.OWNER, v.SEGMENT_NAME, v.BYTES ORIG_SIZE,
       v.INMEMORY_SIZE IN_MEM_SIZE,
       ROUND(v.BYTES / v.INMEMORY_SIZE, 2) COMP_RATIO
FROM   V$IM_SEGMENTS v
ORDER BY 4;
```
