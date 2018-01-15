
- 找回删除的文件

```
删除Oracle datafile
rm /app/oracle/oradata/ORCL/datafile/o1_mf_users_555wrj4o_.dbf

尝试在users表空间中创建表，开始报错
SQL> create table t tablespace users as select * from dual;
create table t tablespace users as select * from dual
                                                 *
ERROR at line 1:
ORA-01116: error in opening database file 4
ORA-01110: data file 4:
'/app/oracle/oradata/ORCL/datafile/o1_mf_users_555wrj4o_.dbf'
ORA-27041: unable to open file
Linux Error: 2: No such file or directory
Additional information: 3


检查dbwr的进程PID
$ ps -ef|grep dbw0|grep -v grep
oracle    2879     1  0 21:38 ?        00:00:00 ora_dbw0_orcl


dbwr会打开所有数据文件的句柄。在proc目录中可以查到，目录名是进程PID，fd表示文件描述符。
$ cd /proc/2879/fd
$ ls -l|grep delete
lrwx------ 1 oracle dba 64 Dec 19 21:50 19 -> /app/oracle/oradata/ORCL/datafile/o1_mf_users_555wrj4o_.dbf (deleted)


直接cp该句柄文件名回原位置。
cp 19 /app/oracle/oradata/ORCL/datafile/o1_mf_users_555wrj4o_.dbf


进行数据文件recover
SQL> alter database datafile 4 offline;

Database altered.

SQL> recover datafile 4;
Media recovery complete.
SQL> alter database datafile 4 online;

Database altered.
```
