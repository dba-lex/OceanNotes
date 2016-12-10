### 创建细粒度策略

```
select 'begin' || chr(10) || 'dbms_fga.add_policy(object_schema     =>''' ||
       owner || ''',' || chr(10) ||
       '                    object_name     =>''' || table_name || '''' || ',' ||
       chr(10) ||
       '                    policy_name     => ''ACCOUNT_ACCESS'',' ||
       chr(10) ||
       '                    statement_types => ''INSERT,UPDATE,DELETE,SELECT'');' ||
       chr(10) || 'end;' || chr(10) || '/'
  from dba_tables
 where owner = 'BLUE';
```

### 删除细粒度策略

```
select 'begin' || chr(10) || 'dbms_fga.drop_policy(object_schema     =>''' ||
       owner || ''',' || chr(10) ||
       '                    object_name     =>''' || table_name || '''' || ',' ||
       chr(10) ||
       '                    policy_name     => ''ACCOUNT_ACCESS'');' ||
       chr(10) || 'end;' || chr(10) || '/'
  from dba_tables
 where owner = 'BLUE';
```


select * from DBA_AUDIT_POLICIES;


select * from dba_common_audit_trail;
