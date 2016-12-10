
## Admin-Managed模式数据库转化为Policy-Manage模式

- Admin-Managed模式

```
oracle$ srvctl status database -d dbalex
Instance dbalex1 is running on node lex-1
Instance dbalex2 is running on node lex-2

ora.dbalex.db
      1        ONLINE  ONLINE       lex-1           Open
      2        ONLINE  ONLINE       lex-2           Open
```

- 获取数据库spfile路径

```
+OCRVOTE/dbalex/spfiledbalex.ora
```

- 停止数据库所有实例

```
srvctl stop database -d dbalex
```

- 创建serverpool（若已存在，则忽略此步骤）

```
grid$ GRID_HOME/bin/srvctl add serverpool -g prodpool -l 2 -u 2 -n lex-1,lex-2
```

- 添加数据库到prodpool

```
oracle$ srvctl add database -d dbalex -c "RAC" -p "+OCRVOTE/dbalex/spfiledbalex.ora" -g prodpool -o /opt/oracle/products/11.2.0
```

- 启动数据库

```
oracle$ srvctl start database -d dbalex
```

- 查看数据库实例运行状态

```
oracle$ srvctl status database -d dbalex
Instance dbalex_1 is running on node lex-1
Instance dbalex_2 is running on node lex-2
```

--------------------------------------------------------------------------------------------
