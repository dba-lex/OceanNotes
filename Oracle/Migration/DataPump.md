# expdp

```
确认主机上有足够的空间容纳导出的数据，并创建一个目录，如mkdir ~/dumpdir
创建directory：  
create directory dumpdir as '/home/oracle/dumpdir';
```

- 全库导出：

```
expdp \'/ as sysdba\' directory=dumpdir job_name=expdp_full dumpfile=expdp_full_%U.dmp logfile=expdp.log parallel=8 \
full=y cluster=n
```

- 限制每个dump文件的最大大小：

```
expdp \'/ as sysdba\' directory=dumpdir job_name=expdp_full dumpfile=expdp_full_%U.dmp logfile=expdp.log parallel=8 \
full=y FILESIZE=10GB cluster=n
```

- 按照表名导出：

```
expdp sgpms_ln/oracle DIRECTORY=dumpdir JOB_NAME=five_impdp DUMPFILE=five_table_%U.dmp logfile=exp_five_table.log parallel=8 \ tables=c_cons,c_vat,c_mp,sa_user,sa_org cluster=n
```

- 按照schema导出：

```
expdp woqu/woqu DIRECTORY=dumpdir JOB_NAME=expdp_schema DUMPFILE=expdp_schema_%U.dmp logfile=expdp_schema.log parallel=8 schemas=woqu cluster=n
```

- 指定PARFILE和QUERY条件：

```
expdp hr/hr parfile=emp_query.par
QUERY=employees:"WHERE department_id > 10 AND salary > 10000"
NOLOGFILE=y
DIRECTORY=dumpdir
DUMPFILE=exp1.dmp
```

- 表空间传输：

```
EXECUTE sys.DBMS_TTS.TRANSPORT_SET_CHECK('WOQU',true);
select * from sys.transport_set_violations;
alter tablespace woqu read only;
expdp system/oracle transport_tablespaces=y tablespaces=(woqu) dumpfile=woqu.dmp logfile=transport_woqu.log
```

- REMAP表空间：

```
impdp sgpms_ln/oracle DIRECTORY=qdatadir LOGFILE=parallel_import.log \
JOB_NAME=idp_two DUMPFILE=two_qdata_%U.dmp \
REMAP_TABLESPACE=USR_PMS_TBS:users,USR_KA_TBS:users,USR_MDA_TBS:users,USR_KA_IDX:users,USR_MDA_IDX:users \
TABLE_EXISTS_ACTION=REPLACE tables=arc_e_pl_amt,e_pl_amt parallel=4
```

- 指定表存在时的行为：

```
TABLE_EXISTS_ACTION={SKIP | APPEND | TRUNCATE | REPLACE}

SKIP leaves the table as is and moves on to the next object. This is not a valid option if the CONTENT parameter is set to DATA_ONLY.
APPEND loads rows from the source and leaves existing rows unchanged.
TRUNCATE deletes existing rows and then loads rows from the source.
REPLACE drops the existing table and then creates and loads it from the source. This is not a valid option if the CONTENT parameter is set to DATA_ONLY.
```

- 指定只导出数据或元信息：

```
expdp woqu/woqu DIRECTORY=dumpdir DUMPFILE=woqu.dmp CONTENT=DATA_ONLY
expdp woqu/woqu DIRECTORY=dumpdir DUMPFILE=woqu.dmp CONTENT=METADATA_ONLY
```
