



- 隐藏参数：

```
set linesize 120
col name for a30
col value for a20
col describ for a60
SELECT x.ksppinm NAME, y.ksppstvl VALUE, x.ksppdesc describ
  FROM SYS.x$ksppi x, SYS.x$ksppcv y
 WHERE x.inst_id = USERENV ('Instance')
   AND y.inst_id = USERENV ('Instance')
   AND x.indx = y.indx
   AND x.ksppinm LIKE '%&par%'
/
```
