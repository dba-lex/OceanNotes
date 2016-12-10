
#### Client-Side Connect Time Failover的含义：

```
如果用户端tnsname 中配置了多个地址
用户发起连接请求时，会先尝试连接地址表中的第一个地址
如果这个连接尝试失败，则继续尝试使用第二个地址，
直至连接成功或者遍历了所有的地址。

这种Failover的特点：　
只在建立连接那一时刻起作用，也就是说，这种Failover方式只在发起连接时才会去感知节点故障，
如果节点没有反应，则自动尝试地址列表中的下一个地址。
一旦连接建立之后，节点出现故障都不会做处理，
从客户端的表现就是会话断开了，用户程序必须重新建立连接。

启用这种Failover的方法就是在客户端的tnsnames.ora中添加FAILOVER=ON 条目，
这个参数默认就是ON，所以即使不添加这个条目，客户端也会获得这种Failover能力。
```

```
RAC =
  (DESCRIPTION =
     (ADDRESS = (PROTOCOL = TCP)(HOST = rac1-vip)(PORT = 1521))
     (ADDRESS = (PROTOCOL = TCP)(HOST = rac2-vip)(PORT = 1521))
     (LOAD_BALANCE=YES)
      (
          CONNECT_DATA=
          (SERVER=DEDICATED)
          (SERVICE_NAME=RAC)
      )
   )
```


从Oracle 8.1.5 版本只有引入了新的Failover 机制—TAF,就是连接建立以后，应用系统运行过程中，如果某个实例发生故障，连接到这个实例上的用户会被自动迁移到其他的健康实例上。
对于应用程序而言，这个迁移过程是透明的，不需要用户的介入，当然，这种透明要是有引导的，因为用户的未提交事务会回滚。
相对与Client-Side Connect Time Failover的用户程序中断会抛出连接错误.

TAF 的配置也很简单，只需要在客户端的tnsnames.ora中添加FAILOVER_MODE配置项。

这个条目有5个子项目需要定义。

- METHOD:

```
用户定义何时创建到其实例的连接，有BASIC 和 PRECONNECT 两种可选值。

BASIC：
是指在感知到节点故障时才创建到其他实例的连接。

PRECONNECT:
是在最初建立连接时就同时建立到所有实例的连接，当发生故障时，立刻就可以切换到其他链路上。

两种方法比较：
BASIC方式在Failover时会有时间延迟，
PRECONNECT方式虽然没有时间延迟，但是建立多个冗余连接会消耗更多资源，
两者就是是用时间换资源和用资源换时间的区别。
```

- TYPE：

```
用于定义发生故障时对完成的SQL 语句如何处理，其中有2种类型：session 和select.
这两种方式对于未提交的事务都会自动回滚，
如果是session模式，则需要重新执行查询语句；
如果是select模式，用户正在执行的select语句会被转移到新的实例上，在新的节点上继续返回后续结果集，而已经返回的记录集则抛弃。

假设用户正在节点1上执行查询，整个结果集共有100条记录，现在已从节点1上返回10条记录，
这时节点1宕机，用户连接被转移到节点2上，会从节点2上继续返回剩下的90条记录，而已经从节点1返回的10条记录不会重复返回给用户，对于用户而言，感受不到这种切换。
显然为了实现select 方式，Oracle 必须为每个session保存更多的内容，包括游标，用户上下文等，需要更多的资源也是用资源换时间的方案。
```

- DELAY 和 RETRIES:

```
这2个参数分别代表重试间隔时间和重试次数。
```

- BACKUP:

```
Specify the failover node by its net service name. A separate net service name must be created for the failover node.
值是tnsnames.ora里另一个别名
```

示例一：TAF with Connect-Time Failover and Client Load Balancing

```
	sales.us.example.com=
	 (DESCRIPTION=
	  (LOAD_BALANCE=on)
	  (FAILOVER=on)
	  (ADDRESS=
	       (PROTOCOL=tcp)  
	       (HOST=sales1-server)  
	       (PORT=1521))
	  (ADDRESS=
	       (PROTOCOL=tcp)  
	       (HOST=sales2-server)  
	       (PORT=1521))
	  (CONNECT_DATA=
	     (SERVICE_NAME=sales.us.example.com)
	     (FAILOVER_MODE=
	       (TYPE=select)
	       (METHOD=basic))))
```

示例二:TAF Retrying a Connection

```
	sales.us.example.com=
	 (DESCRIPTION=
	  (ADDRESS=
	       (PROTOCOL=tcp)  
	       (HOST=sales1-server)  
	       (PORT=1521))
	  (CONNECT_DATA=
	     (SERVICE_NAME=sales.us.example.com)
	     (FAILOVER_MODE=
	       (TYPE=select)
	       (METHOD=basic)
	       (RETRIES=20)
	       (DELAY=15))))
```

示例三:TAF Pre-establishing a Connection

```
	sales1.us.example.com=
	 (DESCRIPTION=
	  (ADDRESS=
	       (PROTOCOL=tcp)  
	       (HOST=sales1-server)  
	       (PORT=1521))
	  (CONNECT_DATA=
	     (SERVICE_NAME=sales.us.example.com)
	     (INSTANCE_NAME=sales1)
	     (FAILOVER_MODE=
	       (BACKUP=sales2.us.example.com)
	       (TYPE=select)
	       (METHOD=preconnect))))
	sales2.us.example.com=
	 (DESCRIPTION=
	  (ADDRESS=
	       (PROTOCOL=tcp)  
	       (HOST=sales2-server)  
	       (PORT=1521))
	  (CONNECT_DATA=
	     (SERVICE_NAME=sales.us.example.com)
	     (INSTANCE_NAME=sales2)
	     (FAILOVER_MODE=
	       (BACKUP=sales1.us.example.com)
	       (TYPE=select)
	       (METHOD=preconnect))))
```

测试示例一:

```
RACDB1 =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 202.0.0.30)(PORT = 1521))
      (ADDRESS = (PROTOCOL = TCP)(HOST = 202.0.0.40)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = RACDB)
      (FAILOVER_MODE=
		(TYPE=select)
		(METHOD=PRECONNECT)
		(RETRIES=180)
		(DELAY=5)
      )
    )
  )
```

SYS@racdb1> select inst_id,username,sid,serial#,STATUS,event,FAILOVER_TYPE,FAILOVER_METHOD,FAILED_OVER,MACHINE from gv$session where username='SCOTT' and MACHINE='ocm1.oracle.com';

   INST_ID USERNAME		    SID    SERIAL# STATUS     EVENT			   FAILOVER_TYPE FAILOVER_M FAI MACHINE
---------- ---------------- ----------- ---------- ---------- ---------------------------- ------------- ---------- --- ------------------
	 1 SCOTT		     21        107 INACTIVE   SQL*Net message from client  NONE 	 NONE	    NO	ocm1.oracle.com
	 1 SCOTT		    141 	99 INACTIVE   SQL*Net message from client  SELECT	 PRECONNECT NO	ocm1.oracle.com

SYS@racdb1>


	SQL> select instance_name from v$instance;

	INSTANCE_NAME
	----------------
	racdb1

	SQL> select * from o1;
	.....
	.....
	ERROR:
	ORA-25401: can not continue fetches



	15315 rows selected.

	SQL> select instance_name from v$instance;

	INSTANCE_NAME
	----------------
	racdb2

	SQL> SQL> select count(*) from o1;

	  COUNT(*)
	----------
	    450640

	SQL>



测试示例二:
[oracle@ocm1 admin]$ cat tnsnames.ora

RACDB1 =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 202.0.0.30)(PORT = 1521))
      (ADDRESS = (PROTOCOL = TCP)(HOST = 202.0.0.40)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = RACDB)
      (FAILOVER_MODE=
		(BACKUP=RACDB2)
		(TYPE=select)
		(METHOD=PRECONNECT)
		(RETRIES=180)
		(DELAY=5)
      )
    )
  )


RACDB2 =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 202.0.0.40)(PORT = 1521))
      (ADDRESS = (PROTOCOL = TCP)(HOST = 202.0.0.30)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = RACDB)
      (FAILOVER_MODE=
                (BACKUP=RACDB1)
                (TYPE=select)
                (METHOD=PRECONNECT)
                (RETRIES=180)
                (DELAY=5)
      )
    )
  )
[oracle@ocm1 admin]$

FAILOVER之前:

	SYS@racdb2> select inst_id,username,sid,serial#,service_name,FAILOVER_TYPE,FAILOVER_METHOD,FAILED_OVER,status,state,event from gv$session where username = 'SCOTT';

	   INST_ID USERNAME		    SID    SERIAL# SERVICE_NAME FAILOVER_TYPE FAILOVER_M FAI STATUS	STATE		    EVENT
	---------- ---------------- ----------- ---------- ------------ ------------- ---------- --- ---------- ------------------- --------------------------------
		 2 SCOTT		    145 	57 racdb	NONE	      NONE	 NO  INACTIVE	WAITING 	    SQL*Net message from client
		 1 SCOTT		     22 	33 racdb	SELECT	      PRECONNECT NO  INACTIVE	WAITING 	    SQL*Net message from client

	SYS@racdb2>

FAILOVER之后:

	SYS@racdb2> select inst_id,username,sid,serial#,service_name,FAILOVER_TYPE,FAILOVER_METHOD,FAILED_OVER,status,state,event from gv$session where username = 'SCOTT';

	   INST_ID USERNAME		    SID    SERIAL# SERVICE_NAME FAILOVER_TYPE FAILOVER_M FAI STATUS	STATE		    EVENT
	---------- ---------------- ----------- ---------- ------------ ------------- ---------- --- ---------- ------------------- --------------------------------
		 2 SCOTT		    145 	57 racdb	SELECT	      PRECONNECT YES INACTIVE	WAITING 	    SQL*Net message from client
		 1 SCOTT		     22 	33 racdb	SELECT	      PRECONNECT NO  INACTIVE	WAITING 	    SQL*Net message from client

	SYS@racdb2>

再次FAILOVER:
SYS@racdb2> select inst_id,username,sid,serial#,service_name,FAILOVER_TYPE,FAILOVER_METHOD,FAILED_OVER,status,state,event from gv$session where username = 'SCOTT';

   INST_ID USERNAME		    SID            SERIAL# SERVICE_NAME FAILOVER_TYPE FAILOVER_M FAI STATUS	STATE		    EVENT
---------- ---------------- ----------- ---------- ------------ ------------- ---------- --- ---------- ---------------------------------------------------
	     2            SCOTT		    145 	    57        racdb	       SELECT PRECONNECT YES INACTIVE	WAITING SQL*Net message from client
	 1 SCOTT		     22 	33 racdb	SELECT	      PRECONNECT NO  INACTIVE	WAITING 	    SQL*Net message from client
	 1 SCOTT		     26 	43 racdb	SELECT	      PRECONNECT YES ACTIVE	WAITED SHORT TIME   SQL*Net message from client

SYS@racdb2>
