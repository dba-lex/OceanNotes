

```
find /opt -name 'alert*.log' -exec cp {} /root/check/ \;
```

- 删除30天前修改过的文件

```
find . -name "*.html" -type f -ctime +30 -exec rm {} \;
```
