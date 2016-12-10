

- 关于insert append的日志生成

```
数据库处于归档模式
当表模式为logging状态时,无论是否使用append模式,都会生成redo.当表模式为nologging状态时,只有append模式,不会生成redo。
数据库处于非归档模式
无论是在logging还是nologing的模式下，append的模式都不会生成redo,而no append模式下都会生成redo。   
```


- 添加日志组或日志成员

```
alter database add logfile thread 1 group 5 size 1G;
ALTER DATABASE ADD LOGFILE MEMBER '/u01/app/oracle/oradata/orcl/redo1.log' TO GROUP 1;
```

- 删除日志成员或日志组

```
ALTER DATABASE DROP LOGFILE MEMBER '$ORACLE_BASE/oradata/u01/logn1.rdo';
ALTER DATABASE DROP LOGFILE GROUP 1;
```
