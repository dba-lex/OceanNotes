从Oracle 10.1.0.3开始引入了利用逻辑备库(logical standby)实施滚动升级(rolling upgrade)的特性；在滚动升级期间(rolling upgrade)，允许主库(primary database)与逻辑备库(logical standby)间运行不同的数据库版本，以最小化宕机时间。

到了11g中增加了可以将物理备库(physical standby)临时性转换成逻辑备库(logical standby)以完成滚动升级，之后将该临时逻辑备库反转为物理备库的功能。使用以上临时转换功能只需要在转换语句”ALTER DATABASE RECOVER TO LOGICAL STANDBY”后加上”KEEP IDENTITY”选项。除去转换细节的区别外，使用物理备库进行滚动升级的过程与10g中的逻辑备库滚动升级没有太大的区别。

这里我们通过实验来亲身体验一下这一新特性，我们假设环境中存在一套11.1.0.7的Data Guard系统：

```
Database Role	DB_UNIQUE_NAME	Version
Primary Database	PROD	11.1.0.7
Physical Standby	SBDB2	11.1.0.7
```

当前的需求是将生产数据库升级到11.2.0.2，但要求最小化应用downtime；这里我们就可以充分利用Data Guard的环境借势升级，同时也不会破坏原有的HA可用性：
1.滚动升级前需要为Primary Database做必要的准备工作，这里如果数据库之前没有启用闪回数据库的话需要enable

```
SQL> SHUTDOWN IMMEDIATE;
SQL> STARTUP MOUNT;
SQL> ALTER DATABASE FLASHBACK ON;

SQL> select flashback_on from v$database;

FLASHBACK_ON
------------------
YES

SQL> ALTER DATABASE OPEN;

/* 同时创建保证还原点以保证升级失败时可以回退 */

SQL> create restore point pre_rolling_upgrd guarantee flashback database;
```

2.将物理备库临时转换为逻辑备库，不同于普通的转换这里要用KEEP IDENTITY

```
SQL> shutdown immediate;
SQL> startup mount;

之后需要在Primary Database中创建Logminer dictionary日志挖掘字典：
SQL> EXECUTE DBMS_LOGSTDBY.BUILD;

以上步骤完成后可以开始将physical standby转换为logical standby了：
SQL> ALTER DATABASE RECOVER TO LOGICAL STANDBY KEEP IDENTITY;

SQL> alter database open;

以下为转换期间的日志:
ALTER DATABASE RECOVER TO LOGICAL STANDBY KEEP IDENTITY
Media Recovery Start: Managed Standby Recovery (SBDB2)
Fast Parallel Media Recovery enabled
Managed Standby Recovery not using Real Time Apply
 parallel recovery started with 2 processes
Media Recovery Waiting for thread 1 sequence 184
2011-03-25 19:36:51.012000 +08:00
Primary database is in MAXIMUM AVAILABILITY mode
Standby controlfile consistent with primary
kcrrvslf: active RFS archival for log 8 thread 1 sequence 188
RFS[3]: Successfully opened standby log 7: '/standby/oradata/SBDB2/onlinelog/o1_mf_7_6qf9b6cw_.log'
2011-03-25 19:40:43.074000 +08:00
Primary database is in MAXIMUM AVAILABILITY mode
Standby controlfile consistent with primary
RFS[3]: Successfully opened standby log 8: '/standby/oradata/SBDB2/onlinelog/o1_mf_8_6qf9b8j4_.log'
Media Recovery Log /standby/flash_recovery_area/SBDB2/archivelog/2011_03_25/o1_mf_1_188_6rrzsqvm_.arc
2011-03-25 19:40:45.459000 +08:00
Media Recovery Log /standby/flash_recovery_area/SBDB2/archivelog/2011_03_25/o1_mf_1_189_6rrzsv10_.arc
Incomplete Recovery applied until change 11881985 time 03/25/2011 19:40:41
Media Recovery Complete (SBDB2)
tkcrrxms: Killing 140733193388036 processes (all RFS)
2011-03-25 19:40:48.011000 +08:00
Begin: Standby Redo Logfile archival
End: Standby Redo Logfile archival
RESETLOGS after incomplete recovery UNTIL CHANGE 11881985
Resetting resetlogs activation ID 157033137 (0x95c22b1)
Online log /standby/oradata/SBDB2/onlinelog/o1_mf_1_6qf99xt6_.log: Thread 1 Group 1 was previously cleared
Online log /standby/flash_recovery_area/SBDB2/onlinelog/o1_mf_1_6qf99y1p_.log: Thread 1 Group 1 was previously cleared
Online log /standby/oradata/SBDB2/onlinelog/o1_mf_2_6qf9b01v_.log: Thread 1 Group 2 was previously cleared
Online log /standby/flash_recovery_area/SBDB2/onlinelog/o1_mf_2_6qf9b06y_.log: Thread 1 Group 2 was previously cleared
Online log /standby/oradata/SBDB2/onlinelog/o1_mf_3_6qf9b21o_.log: Thread 1 Group 3 was previously cleared
Online log /standby/flash_recovery_area/SBDB2/onlinelog/o1_mf_3_6qf9b268_.log: Thread 1 Group 3 was previously cleared
Standby became primary SCN: 11881983
Setting recovery target incarnation to 4
Converting standby mount to primary mount.
ACTIVATE STANDBY: Complete - Database mounted as primary (SBDB2)
Completed: ALTER DATABASE RECOVER TO LOGICAL STANDBY KEEP IDENTITY

我们还需要完成一系列逻辑备库的配置:

a.禁止在逻辑备库端删除外籍日志

SQL> EXECUTE DBMS_LOGSTDBY.APPLY_SET('LOG_AUTO_DELETE', 'FALSE');
BEGIN DBMS_LOGSTDBY.APPLY_SET('LOG_AUTO_DELETE', 'FALSE'); END;

ERROR at line 1:
ORA-06550: line 1, column 7:
PLS-00201: identifier 'DBMS_LOGSTDBY.APPLY_SET' must be declared
ORA-06550: line 1, column 7:
PL/SQL: Statement ignored

/* 缺少DBMS_LOGSTDBY包，可以从以下脚本创建 */

SQL> @?/rdbms/admin/dbmslsby.sql

SQL> EXECUTE DBMS_LOGSTDBY.APPLY_SET('LOG_AUTO_DELETE', 'FALSE');
PL/SQL procedure successfully completed.

b.执行以下语句可以将逻辑备库不支持的而又在主库上运行过的事务记录到DBA_LOGSTDBY_EVENTS中

SQL> EXECUTE DBMS_LOGSTDBY.APPLY_SET('MAX_EVENTS_RECORDED',DBMS_LOGSTDBY.MAX_EVENTS);
PL/SQL procedure successfully completed.

c.启动在逻辑备库上的SQL APPLY:

SQL> ALTER DATABASE START LOGICAL STANDBY APPLY IMMEDIATE;
Database altered.
```

3.在步骤3中我们实际实施对逻辑备库的升级操作

```
1).在逻辑备库上停止SQL APPLY

SQL> ALTER DATABASE STOP LOGICAL STANDBY APPLY;

2).升级逻辑备库(upgrade logical standby)

SQL> shutdown immediate;

[maclean@rh6 admin]$ cp $ORACLE_HOME/dbs/spfileSBDB2.ora /s01/product/11.2.0/dbhome_2/dbs
[maclean@rh6 dbs]$ orapwd file=orapwSBDB2 password=XXXXX entries=10

/* 将原SBDB2实例的spfile复制到新的11.2.0.2版本的ORACLE_HOME下，并在新的ORACLE_HOME/dbs目录下创建
   与主库一致的密码文件 */

[maclean@rh6 ~]$ source 11gr2env.sh

[maclean@rh6 ~]$ export ORACLE_SID=SBDB2

SQL> startup upgrade;

SQL> @?/rdbms/admin/catupgrd
..................
/* 确认升级数据字典成功，值得一提的是11g的数据字典升级工作极度复杂，因此该阶段耗费大量时间 */

3)在逻辑备库上启用SQL APPLY

SQL>  ALTER DATABASE START LOGICAL STANDBY APPLY IMMEDIATE;

SQL> ALTER SESSION SET NLS_DATE_FORMAT = 'DD-MON-YY HH24:MI:SS';
Session altered.

SQL> SELECT SYSDATE, APPLIED_TIME FROM V$LOGSTDBY_PROGRESS;
SYSDATE 	   APPLIED_TIME
------------------ ------------------
25-MAR-11 21:26:26 25-MAR-11 21:26:21

/* 监控以上SQL APPLY进度，是否与当前时间接近 */

4)审查升级过程中的监控事件，可以直接查询DBA_LOGSTDBY_EVENTS视图，
该视图记录了在逻辑备库上没有applied的DDL与DML操作

SQL> SET LONG 1000
SQL> SET PAGESIZE 180
SQL> SET LINESIZE 79
SQL> SELECT EVENT_TIMESTAMP, EVENT, STATUS FROM DBA_LOGSTDBY_EVENTS ORDER BY EVENT_TIMESTAMP;

EVENT_TIMESTAMP
---------------------------------------------------------------------------
EVENT
-------------------------------------------------------------------------------
STATUS
-------------------------------------------------------------------------------
25-MAR-11 07.53.39.593831 PM

ORA-16111: log mining and apply setting up

25-MAR-11 07.53.39.594478 PM

Apply LWM 11881984, HWM 11881984, SCN 11881984

25-MAR-11 07.57.54.467786 PM

Shutdown acknowledged

25-MAR-11 07.57.57.516431 PM

ORA-16128: User initiated stop apply successfully completed

25-MAR-11 09.06.42.150573 PM

ORA-16111: log mining and apply setting up

25-MAR-11 09.06.42.345581 PM

Apply LWM 11883610, HWM 11883615, SCN 11883614

25-MAR-11 09.12.56.429392 PM

ORA-16128: User initiated stop apply successfully completed

25-MAR-11 09.26.16.124581 PM

ORA-16111: log mining and apply setting up

25-MAR-11 09.26.16.145925 PM

Apply LWM 11883610, HWM 11883615, SCN 11883614

25-MAR-11 09.26.20.723913 PM
ALTER DATABASE OPEN
ORA-16226: DDL skipped due to lack of support

ORA-16226错误表示相关DDL操作在逻辑备库上不受支持，常见的原因是该类操作涉及内部对象
ORA-16129错误表示相关DML操作在逻辑备库上不受支持

5)开始主备库之间的切换(switchover)

ALTER DATABASE COMMIT TO SWITCHOVER TO LOGICAL STANDBY;

SQL> SELECT SWITCHOVER_STATUS FROM V$DATABASE;

SWITCHOVER_STATUS
--------------------
TO PRIMARY

SQL> ALTER DATABASE COMMIT TO SWITCHOVER TO PRIMARY;

Database altered.

以上切换完成后当前主库(Primary Database)的日志将无法再传输到逻辑备库(PROD)中，当前主库SBDB的版本为11.2.0.2，
而逻辑备库RPOD(原主库)仍为11.1.0.7；因为当前主库的日志都无法传送到低版本的logical standby中，这意味着当前主库是不受
任何保护的状态，这是滚动升级中无法避免的一个真空期。
现在我们可以在原逻辑备库上启动用户应用程序了。
```

4.将原主库(PROD)闪回到之前创建的还原点上

```
SQL> SHUTDOWN IMMEDIATE;
SQL> STARTUP MOUNT;

SQL> select name from v$restore_point;

NAME
--------------------------------------------------------------------------------
PRE_ROLLING_UPGRD

SQL> flashback database to restore point pre_rolling_upgrd;
```

5.以新的ORACLE_HOME binary加载原主库(PROD)实例

```
SQL> shutdown immediate;
/* 先将原主库(PROD)实例关闭  */

[maclean@rh6 dbs]$ cp /standby/product/11.1.0/db_3/dbs/spfilePROD.ora $ORACLE_HOME/dbs
[maclean@rh6 dbs]$ orapwd file=orapwPROD password=XXXXX entries=10

/* 在新版本的ORACLE_HOME下复制spfile并重建密码文件 */

[maclean@rh6 dbs]$ export ORACLE_SID=PROD

SQL> startup mount;

/* 这里我们不需要运行升级脚本，后面的redo apply会自动升级PROD上的数据字典 */
```

6.将原主库(PROD)切换会物理备库(physical standby)

```
SQL> ALTER DATABASE CONVERT TO PHYSICAL STANDBY;
SQL> SHUTDOWN IMMEDIATE;
```

7.启动原主库(PROD)当前物理备库上的介质恢复进程

```
SQL> STARTUP MOUNT;
SQL>  ALTER DATABASE RECOVER MANAGED STANDBY DATABASE
USING CURRENT LOGFILE DISCONNECT FROM SESSION;
```

8.在主库与物理备库间实施切换

```
SBDB2 switchover回物理备库状态
SQL>  alter database commit to switchover to physical standby;

PROD切换为Primary Database
SQL> alter database commit to switchover to primary;

因为数据字典升级期间产生了大量的重做日志，切换之前需要完成对这些redo的应用，
所以该阶段将耗费大量时间，可以从PROD的告警日志中看到类似下面的内容:

Media Recovery Log /standby/arch/1_51_746739648.dbf
2011-03-26 00:32:09.625000 +08:00
Media Recovery Log /standby/arch/1_52_746739648.dbf
2011-03-26 00:32:16.931000 +08:00
Media Recovery Log /standby/arch/1_53_746739648.dbf
2011-03-26 00:32:23.149000 +08:00
Media Recovery Log /standby/arch/1_54_746739648.dbf
2011-03-26 00:32:31.732000 +08:00
Media Recovery Log /standby/arch/1_55_746739648.dbf
2011-03-26 00:32:43.807000 +08:00
Media Recovery Log /standby/arch/1_56_746739648.dbf
```

9.出于节约磁盘空间的考虑将PROD上最初建立的还原点删除：

```
SQL> DROP RESTORE POINT PRE_ROLLING_UPGRD
```
