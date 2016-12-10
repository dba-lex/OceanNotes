
```
#!/bin/bash
#------------------------------------------------------------------
# File Name    : user_ddl.sh
# Author       : Lex Guo
# Mail         : dbalex@dbalex.com
# Description  : Displays the DDL for a specific user.
# Call Syntax  : sh user_ddl.sh <USERNAME>
# Last Modified: 2016-06-05
#------------------------------------------------------------------

ORACLE_SID=qdata1
NLS_LANG=AMERICAN_AMERICA.ZHS16GBK; export NLS_LANG
ORACLE_BASE=/opt/oracle
ORACLE_HOME=$ORACLE_BASE/products/11.2.0
PATH=$PATH:$ORACLE_HOME/bin:$ORACLE_HOME/oracm/bin:$ORACLE_HOME/OPatch:$ORACLE_HOME/jdbc
LD_LIBRARY_PATH=$ORACLE_HOME/lib:$ORACLE_HOME/ctx/lib:$ORACLE_HOME/oracm/lib:/usr/lib
CLASSPATH=$ORACLE_HOME/JRE:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib:$ORACLE_HOME/network/jlib:$ORACLE_HOME/jdbc/lib
export ORACLE_BASE ORACLE_HOME ORACLE_SID PATH LD_LIBRARY_PATH CLASSPATH
NLS_DATE_FORMAT="yyyy-mm-dd hh24:mi:ss";export NLS_DATE_FORMAT

NAME="$1"
sqlplus -S / as sysdba <<EOF
set long 20000 longchunksize 20000 pagesize 0 linesize 1000 feedback off verify off trimspool on
column ddl format a1000

begin
   dbms_metadata.set_transform_param (dbms_metadata.session_transform, 'SQLTERMINATOR', true);
   dbms_metadata.set_transform_param (dbms_metadata.session_transform, 'PRETTY', true);
end;
/

spool '${NAME}_CREATE.SQL'
SELECT DBMS_METADATA.GET_DDL('USER', '$NAME') DDL FROM DUAL
union all
select dbms_metadata.get_ddl('ROLE', r.role) AS ddl
from   dba_roles r
where  r.role = upper('$NAME')
union all
select dbms_metadata.get_granted_ddl('ROLE_GRANT', rp.grantee) AS ddl
from   dba_role_privs rp
where  rp.grantee = upper('$NAME')
and    rownum = 1
union all
select dbms_metadata.get_granted_ddl('SYSTEM_GRANT', sp.grantee) AS ddl
from   dba_sys_privs sp
where  sp.grantee = upper('$NAME')
and    rownum = 1
union all
select dbms_metadata.get_granted_ddl('OBJECT_GRANT', tp.grantee) AS ddl
from   dba_tab_privs tp
where  tp.grantee = upper('$NAME')
and    rownum = 1
/
spool off
set linesize 80 pagesize 14 feedback on verify on
EOF
```
