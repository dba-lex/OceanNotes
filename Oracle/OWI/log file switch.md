1. log file switch (archiving needed)

```
这个等待事件出现时通常是因为日志组循环写满以后，第一个日志归档尚未完成，出现该等待。出现该等待，可能表示io 存在问题。解决办法:
*可以考虑增大日志文件和增加日志组
*移动归档文件到快速磁盘
*调整log_archive_max_processes .
```

2. log file switch (checkpoint incomplete)-日志切换(检查点未完成)

```
当你的日志组都写完以后，LGWR 试图写第一个log file，如果这时数据库没有完成写出记录在第一个log file 中的dirty 块时(例如第一个检查点未完成)，该等待事件出现。
该等待事件通常表示你的DBWR 写出速度太慢或者IO 存在问题。
为解决该问题，你可能需要考虑增加额外的DBWR 或者增加你的日志组或日志文件大小或者解决磁盘IO问题。
```
