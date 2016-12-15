# rman参考手册

### format string:

```
%c  The copy number of the backup piece within a set of duplexed backup pieces. If you did not duplex a backup, then this variable is 1 for backup sets and 0 for proxy copies.If one of these commands is enabled, then the variable shows the copy number. The maximum value for %c is 256.

%d  The name of the database.

%D  The current day of the month (in format DD)

%F  Combination of DBID, day, month, year, and sequence into a unique and repeatable generated name.

%M  The month (format MM)

%n  The name of the database, padded on the right with x characters to a total length of eight characters. (AKA: Porn star alias name) For example, if the scott is the database name, %n= scottxxx.

%p  The piece number within the backup set. This value starts at 1 for each backup set and is incremented by 1 as each backup piece is created. Note: If you specify PROXY, then the %p variable must be included in the FORMAT string either explicitly or implicitly within %U.

%s  The backup set number. This number is a counter in the control file that is incremented for each backup set. The counter value starts at 1 and is unique for the lifetime of the control file. If you restore a backup control file, then duplicate values can result.Also, CREATE CONTROLFILE initializes the counter back to 1.

%t  The backup set time stamp, which is a 4-byte value derived as the number of seconds elapsed since a fixed reference time.The combination of %s and %t can be used to form a unique name for the backup set.

%T  The year, month, and day (YYYYMMDD)

%u  An 8-character name constituted by compressed representations of the backup set number and the time the backup set was created.

%U  A convenient shorthand for %u_%p_%c that guarantees uniqueness in generated backup filenames.If you do not specify a format, RMAN uses %U by default.

%Y  The year (YYYY)

%%  Specifies the '%' character. e.g. %%Y translates to %Y.
```

### configure举例

```
configure default device type to disk;
configure device type disk parallelism 3;

configure channel DEVICE TYPE DISK maxpiecesize 100M;
configure channel DEVICE TYPE DISK format '/u01/app/oracle/backup/%d_%I_%s_%p_%T_%u.bak';

configure controlfile autobackup on;
CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '/u01/app/oracle/backup/CTL_%F.bak';

configure retention policy to recovery window of 7 days;
```

### backup command

- database level

```
RMAN> backup database;
RMAN> backup database tag='full_bak1';
RMAN> backup database plus archivelog;
RMAN> backup as compressed backupset database;
RMAN> backup as backupset format '/BACKUP/%d_%I_%s_%p_%T_%u.FUL' database;

RMAN> backup database keep until time='sysdate+30';
RMAN> backup database skip readonly;
RMAN> backup database skip offline;
RMAN> backup database skip inAccessible;
RMAN> backup database ship readonly skip offline ship inaccessible;

RMAN> backup copy of database;

RMAN> backup database plus archivelog delete all input;

RMAN> backup incremental level 0 database;
RMAN> backup incremental level 1 database;
RMAN> backup incremental level 1 cumulative database;
```

- datafile level

```
RMAN> backup tag='month_full_backup' datafile 1, 2, 3, 4;
RMAN> backup as copy datafile 15 format '/oradata/newdatafile15.dbf';
```

- skip

```
backup database skip readonly;
backup database skip offline;
backup database skip inaccessible;
backup database ship readonly skip offline ship inaccessible;
```

- tablespace level

```
RMAN> backup as backupset format '/BACKUP/df_%d_%s_%p.bus' tablespace users;
```

- archivelog level

```
RMAN> BACKUP ARCHIVELOG SEQUENCE 32 thread 1;
RMAN> BACKUP ARCHIVELOG ALL;
RMAN> backup archivelog from scn 4172310;
RMAN> backup archivelog from sequence 144 thread 1;
RMAN> backup archivelog sequence between 144 and 146 thread 1;
RMAN> backup archivelog until sequence 145 THREAD 1;
RMAN> backup archivelog from time '2016-07-18 23:00:00';
RMAN> BACKUP ARCHIVELOG UNTIL TIME 'SYSDATE-5';
RMAN> BACKUP ARCHIVELOG FROM TIME 'SYSDATE-30' UNTIL TIME 'SYSDATE-7';

RMAN> backup XXX plus archivelog delete input all;
RMAN> backup XXX plus archivelog delete all input;

With DELETE INPUT, RMAN only deletes the specific copy of the archived redo log chosen for the backup set. With DELETE ALL INPUT, RMAN will delete each backed-up archived redo log file from all log archiving destinations.
```

- EMC datadomain使用一例

```
rman target / nocatalog msglog /home/oracle/arch0926/backup.log append <<EOF
run {
allocate channel d1 type 'SBT_TAPE' PARMS
'SBT_LIBRARY=/app/oracle/rdbms/product/mes11204/lib/libddobk.so, ENV=(STORAGE_UNIT=oracle,BACKUP_HOST=dd2500.mflex.com.cn, ORACLE_HOME=/app/oracle/rdbms/product/mes11204)';
allocate channel d2 type 'SBT_TAPE' PARMS
'SBT_LIBRARY=/app/oracle/rdbms/product/mes11204/lib/libddobk.so, ENV=(STORAGE_UNIT=oracle,BACKUP_HOST=dd2500.mflex.com.cn, ORACLE_HOME=/app/oracle/rdbms/product/mes11204)';
allocate channel d3 type 'SBT_TAPE' PARMS 'SBT_LIBRARY=/app/oracle/rdbms/product/mes11204/lib/libddobk.so, ENV=(STORAGE_UNIT=oracle,BACKUP_HOST=dd2500.mflex.com.cn, ORACLE_HOME=/app/oracle/rdbms/product/mes11204)';

send 'set username ddboost password xxxx servername dd2500.mflex.com.cn';

set archivelog destination to '/tmp/odsbak/arch';
restore archivelog from logseq=853 thread=2;
release channel d1;
release channel d2;
release channel d3;
}
EOF
```

### copy command

```
copy datafile 5 to '/backup/df_5.dbf';
copy current controlfile to 'c:demo.ctl';
```

### crosscheck

```
RMAN> crosscheck backup             核对所有备份集;   
RMAN> crosscheck backup of database      核对所有数据文件的备份集;   
RMAN> crosscheck backup of tablespace users      核对特定表空间的备份集;   
RMAN> crosscheck backup of datafile 4    核对特定数据文件的备份集;   
RMAN> crosscheck backup of controlfile   核对控制文件的备份集;   
RMAN> crosscheck backup of spfile    核对SPFILE的备份集;   
RMAN> crosscheck backup of archivelog sequence 3 核对归档日志的备份集;   
RMAN> crosscheck copy               核对所有映像副本;   
RMAN> crosscheck copy of database       核对所有数据文件的映像副本;   
RMAN> crosscheck copy of tablespace users       核对特定表空间的映像副本;   
RMAN> crosscheck copy of datafile 6        核对特定数据文件的映像副本;   
RMAN> crosscheck copy of archivelog sequence 4  核对归档日志的映像副本;   
RMAN> crosscheck copy of controlfile       核对控制文件的映像副本;  
RMAN> crosscheck backup tag='SAT_BACKUP';
RMAN> crosscheck backup completed after 'sysdate - 2'
RMAN> crosscheck backup completed between 'sysdate - 5' and 'sysdate -2 '
RMAN> crosscheck backup device type sBT;
RMAN> crosscheck archivelog all;
RMAN> crosscheck archivelog like '%ARC00012.001'
RMAN> crosscheck archivelog from sequence 12;
RMAN> crosscheck archivelog until sequence 522;
```


### report commands

```
RMAN> report schema;
RMAN> report need backup incremental 0 database;
RMAN> report need backup days 3;
RMAN> report need backup redundancy 2;
RMAN> report need backup recovery window of 3 days;
RMAN> report obsolete redundancy 2;
RMAN> report obsolete recovery window of 3 days;
```


### delete command

```
RMAN> delete noprompt expired copy;
RMAN> delete noprompt expired backup;
RMAN> delete noprompt obsolete;
RMAN> delete obsolete redundancy 2;
RMAN> delete obsolete recovery window of 3 days;
RMAN> delete backupset 4;
```


### LIST command

```
RMAN> list backup of database summary;
RMAN> list backup of tablespace system summary;
RMAN> list backup of datafile {n|'file_name'} summary;
RMAN> list backup of database by {file|backup};
RMAN> list copy of database;

list backup of archivelog all;
list backup of archivelog from scn ...;
list backup of archivelog until scn ...;
list backup of archivelog from sequence ..;
list backup of archivelog until time 'sysdate-10';
list backup of archivelog {all, from, high, like, logseq, low, scn, sequence, time, until};

list incarnation;
```


### block change tracking

```
SQL> alter database enable block change tracking using file '/mydir/rman_change_track.f' reuse;
SQL> alter database disable block change tracking;
```



### RMAN v$ views

```
V$ARCHIVED_LOG
V$BACKUP_CORRUPTION
V$BACKUP_DEVICE
V$BACKUP_FILES
V$BACKUP_PIECE
V$BACKUP_REDOLOG
V$BACKUP_SET
V$BACKUP_SPFILE
V$COPY_CORRUPTION
V$RMAN_CONFIGURATION
```

### 全库备份示例

```
run{
allocate channel c1 type disk MAXPIECESIZE 8192M ;
allocate channel c2 type disk MAXPIECESIZE 8192M ;
allocate channel c3 type disk MAXPIECESIZE 8192M ;
allocate channel c4 type disk MAXPIECESIZE 8192M ;
allocate channel c5 type disk MAXPIECESIZE 8192M ;
allocate channel c6 type disk MAXPIECESIZE 8192M ;
sql 'alter system archive log current';
backup as compressed backupset format '/backup/%d_%T_%p_%s_%u.ful'  database;
sql 'alter system archive log current';
backup as compressed backupset  format '/backup/%d_%T_%p_%s_%u.arb' archivelog all;
backup current controlfile  format '/backup/%d_%T_%p_%s_%u.cbk' channel c3;

release channel c1;
release channel c2;
release channel c3;
release channel c4;
release channel c5;
release channel c6;
}
```
