
- version

```
select * from v$version;
select * from  dba_registry_history;
```

- 字符集

```
col PROPERTY_NAME for a50
col PROPERTY_VALUE for a50
set lines 400
select PROPERTY_NAME,PROPERTY_VALUE from database_properties
where PROPERTY_NAME in ('NLS_CHARACTERSET','NLS_NCHAR_CHARACTERSET');
```
