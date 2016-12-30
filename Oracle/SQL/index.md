
- 手工迁移索引

```
select 'create ' ||
                   decode(t.uniqueness, 'UNIQUE', 'UNIQUE INDEX ', 'INDEX ') || '"' ||
                   t.owner || '"' || '.' || '"' || t.INDEX_NAME || '"' ||
                   ' ON ' || '"' || t.TABLE_OWNER || '"' || '.' || '"' ||
                   t.TABLE_NAME || '"' || '(' || b.COL || ') ' ||
                   decode(t.partitioned, 'NO', '', 'YES', 'LOCAL ') ||
                   decode(t.temporary,
                          'Y',
                          '',
                          ' tablespace ' ||
                          nvl(t.TABLESPACE_NAME, 'FMISCOMP ')) ||
                   decode(t.temporary, 'Y', '', ' NOLOGGING PARALLEL 16 ') STR
              from dba_indexes t,
                   (select index_owner,
                           index_name,
                           to_char(wm_concat('"'||COLUMN_NAME||'"')) col
                      from (select index_owner, index_name, COLUMN_NAME
                              from dba_ind_columns
                             order by index_name, column_position)
                     group by index_owner, index_name) b
             where T.index_name = b.index_name
               and T.owner = b.index_owner
               AND T.table_name NOT LIKE 'TMP%'
               AND T.table_name NOT LIKE 'TEMP%'
               and T.owner IN (SELECT username from system.musers);
```
