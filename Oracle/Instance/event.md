
- 10046

```
session 级别：　alter session set events ‘10046  trace name context forever,level X’;
system 级别 :      alter system  set events ‘10046  trace name context forever,level X’;

针对非本会话的 某一个进程设置，如果你知道他的SPID 操作系统进程号
oradebug setospid SPID;
oradebug event 10046 trace name context forever, level X;
```
