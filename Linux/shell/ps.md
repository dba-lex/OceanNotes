
- 批量杀进程

```
ps -ef |grep grid|grep -v grep |awk '{print $2}'|xargs kill -9
```
