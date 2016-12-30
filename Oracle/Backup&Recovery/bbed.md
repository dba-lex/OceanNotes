
### 编译

- 10g

```
[oracle@dbalex lib]$ pwd
/u01/app/oracle/product/10.2.0/rdbms/lib
[oracle@dbalex lib]$ make -f ins_rdbms.mk BBED=$ORACLE_HOME/bin/bbed $ORACLE_HOME/bin/bbed
chmod u+x $ORACLE_HOME/bin/bbed
```

- 11g

```
Oracle Database 11g中缺省的未提供BBED库文件，但是可以用10g的文件编译出来
Copy $ORA10g_HOME/rdbms/lib/ssbbded.o to $ORA11g_HOME/rdbms/lib
Copy $ORA10g_HOME/rdbms/lib/sbbdpt.o to $ORA11g_HOME/rdbms/lib

Copy $ORA10g_HOME/rdbms/mesg/bbedus.msb to $ORA11g_HOME/rdbms/mesg
Copy $ORA10g_HOME/rdbms/mesg/bbedus.msg to $ORA11g_HOME/rdbms/mesg
Copy $ORA10g_HOME/rdbms/mesg/bbedar.msb to $ORA11g_HOME/rdbms/mesg

[oracle@dbalex lib]$ pwd
/u01/app/oracle/product/10.2.0/rdbms/lib
[oracle@dbalex lib]$ make -f ins_rdbms.mk BBED=$ORACLE_HOME/bin/bbed $ORACLE_HOME/bin/bbed
chmod u+x $ORACLE_HOME/bin/bbed
```

BBED的缺省口令为blockedit


### 修改数据文件scn

- 将表空间offline，immediate方式的offline在online时需要进行介质恢复

```
sys@ORCL>alter tablespace lex offline immediate;

Tablespace altered.
```

- 切换日志，并删除归档

- 试图将表空间online

```
sys@ORCL>alter tablespace lex online;
alter tablespace lex online
*
ERROR at line 1:
ORA-01113: file 14 needs media recovery
ORA-01110: data file 14: '/opt/oracle/oracle/product/10.2.0/oradata/orcl/lex01.dbf'
```

到这里表空间在online时需要介质恢复，但是归档已经删掉了。

表空间online时，数据库会校验数据文件头。

所以初步方案：考虑利用bbed工具修改14号文件的数据文件头，骗过Oracle的启动校验过程。

- copy offline的数据文件及一个正常online的数据文件

```
RMAN> copy datafile '/opt/oracle/oracle/product/10.2.0/oradata/orcl/users01.dbf' to '/home/oracle/users01.dbf';
RMAN> copy datafile '/opt/oracle/oracle/product/10.2.0/oradata/orcl/lex01.dbf' to '/home/oracle/lex01.dbf';

/opt/oracle/oracle/product/10.2.0/oradata/orcl/lex01.dbf是offline的数据文件
```

- 编辑文件列表

```
select name,bytes from v$datafile;

$cat file.txt
1 /home/oracle/users01.dbf       27525120
2 /home/oracle/lex01.dbf	 20971520
```

- 编辑参数文件

```
$cat bbed.par
blocksize=8192
listfile=/home/oracle/file.txt
mode=edit
```

- 启动bbed

```
$bbed parfile=bbed.par
```

- 数据文件头介绍

```
数据文件头位于数据文件的第一个block，bbed可以检验和查看block的信息。数据文件头包含一个独立的结构体-kcvfh，Oracle在校验一个文件是否与其他文件同步时，会查看结构体中的四个属性。

a.kscnbas : SCN of last change to the datafile.
b.kcvcptim : Time of the last change to the datafile.
c.kcvfhcpc : Checkpoint count.
d.kcvfhccc : Unknown,but is always 1 less than checkpoint count.

scn：包含kscnbas,kscnwrp
kcvcptim：最后改变时间
kcvfhcpc：检查点计数
kcvfhccc：永远比检查点计数小一
kcrbaseq：日志序列号     
kcrbabno：日志块号    
kcrbabof：

OK,我们的任务就是用modify命令把offline的数据文件头的四个值修改为正常online的数据文件头的对应值。
```

- 修改scn

```
BBED> set dba 1,1
DBA 0×00400001 (4194305 1,1)

BBED> p kcvfhckp
struct kcvfhckp, 36 bytes @484
struct kcvcpscn, 8 bytes @484
ub4 kscnbas @484 0x3111afd5
ub2 kscnwrp @488 0×0000
ub4 kcvcptim @492 0×32891563
ub2 kcvcpthr @496 0×0001
union u, 12 bytes @500
struct kcvcprba, 12 bytes @500
ub4 kcrbaseq @500 0x000029fd
ub4 kcrbabno @504 0x0000038d
ub2 kcrbabof @508 0×0010
ub1 kcvcpetb[0] @512 0×02
ub1 kcvcpetb[1] @513 0×00
ub1 kcvcpetb[2] @514 0×00
ub1 kcvcpetb[3] @515 0×00
ub1 kcvcpetb[4] @516 0×00
ub1 kcvcpetb[5] @517 0×00
ub1 kcvcpetb[6] @518 0×00
ub1 kcvcpetb[7] @519 0×00

BBED> p kcvfhcpc
ub4 kcvfhcpc @140 0x00002a46

BBED> p kcvfhccc
ub4 kcvfhccc @148 0x00002a45

BBED> modify /x 3111afd5 dba 2,1 offset 484
BBED> modify /x 0000 dba 2,1 488
BBED> modify /x 32891563 dba 2,1 offset 492
BBED> modify /x 00002a46 dba 2,1 offset 140
BBED> modify /x 00002a45 dba 2,1 offset 148
BBED> modify /x 000029fd dba 2,1 offset 500
BBED> modify /x 0000038d dba 2,1 offset 504
BBED> modify /x 0010 dba 2,1 offset 508

如遇到如下报错：
BBED> modify /x 3111afd5 dba 2,1 484
BBED-00209: invalid number (3111afd5)
遇到这种情况，我们两位一起改：
BBED> modify /x 3111 dba 2,1 484
BBED> modify /x afd5 dba 2,1 486

BBED> sum dba 2,1 apply
```

- 重建控制文件

```
控制文件中14号数据文件的SCN没有修改，所以数据文件头的SCN高于控制文件中的SCN。这时候需要重建控制文件。
```

- recover

```
sys@ORCL>recover database using backup controlfile;
ORA-00279: change 823243740 generated at 05/18/2014 01:08:24 needed for thread 1
ORA-00289: suggestion : /opt/oracle/archive/1_10752_840086102.arc
ORA-00280: change 823243740 for thread 1 is in sequence #10752

Specify log: {<RET>=suggested | filename | AUTO | CANCEL}
/opt/oracle/oracle/product/10.2.0/oradata/orcl/redo12.log
Log applied.
Media recovery complete.
```

- 打开数据库

```
sys@ORCL>alter database open resetlogs;

Database altered.

sys@ORCL>alter tablespace lex online;

Tablespace altered.
```
