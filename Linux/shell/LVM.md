

扩展文件系统：

```
扩大逻辑卷：lvresize -L +40G /dev/vg_db/lv_root
更改文件系统大小
resize2fs -p /dev/vg_db/lv_root
查看修改结果: lvs / df -h
```
