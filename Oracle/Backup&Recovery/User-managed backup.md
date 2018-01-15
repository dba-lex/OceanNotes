
# User-Managed backup

You must put a read/write tablespace in backup mode to make user-managed datafile backups when the tablespace is online and the database is open. The ALTER TABLESPACE ... BEGIN BACKUP statement places a tablespace in backup mode. In backup mode, the database copies whole changed data blocks into the redo stream. After you take the tablespace out of backup mode with the ALTER TABLESPACE ... END BACKUP or ALTER DATABASE END BACKUP statement, the database advances the datafile checkpoint SCN to the current database checkpoint SCN.

When restoring a datafile backed up in this way, the database asks for the appropriate set of redo log files to apply if recovery is needed. The redo logs contain all changes required to recover the datafiles and make them consistent.

If you do not use BEGIN BACKUP to mark the beginning of an online tablespace backup and wait for this statement to complete before starting your copies of online tablespaces, then the datafile copies produced are not usable for subsequent recovery operations. Attempting to recover such a backup is risky and can return errors that result in inconsistent data. For example, the attempted recovery operation can issue a fuzzy file warning, and can lead to an inconsistent database that you cannot open.

- fuzzy file

A datafile that contains at least one block with an SCN greater than or equal to the checkpoint SCN in the datafile header. Fuzzy files are possible because database writer does not update the SCN in the file header with each file block write. For example, this situation occurs when Oracle updates a datafile that is in backup mode. A fuzzy file that is restored always requires media recovery.


```
SQL> alter database begin backup;

Database altered.

SQL> SELECT * FROM V$BACKUP WHERE STATUS = 'ACTIVE';

     FILE# STATUS		 CHANGE# TIME	       CON_ID
---------- ------------------ ---------- --------- ----------
	 1 ACTIVE		 1678367 11-JAN-18	    1
	 3 ACTIVE		 1678367 11-JAN-18	    1
	 4 ACTIVE		 1678367 11-JAN-18	    1
	 7 ACTIVE		 1678367 11-JAN-18	    1
	 9 ACTIVE		 1678367 11-JAN-18	    3
	10 ACTIVE		 1678367 11-JAN-18	    3
	11 ACTIVE		 1678367 11-JAN-18	    3
	12 ACTIVE		 1678367 11-JAN-18	    3

8 rows selected.

SQL> alter database end backup;

Database altered.

SQL> SELECT * FROM V$BACKUP WHERE STATUS = 'ACTIVE';

no rows selected
```

### 基于存储快照技术的备份

Supported Backup, Restore and Recovery Operations using Third Party Snapshot Technologies (文档 ID 604683.1)

https://docs.oracle.com/en/database/oracle/oracle-database/12.2/bradv/user-managed-flashback-dbpitr.html#GUID-5A3BB8EE-291F-41C6-976C-1F304069FF0B
