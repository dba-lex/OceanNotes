


- 添加数据库资源到crs集群

```
srvctl add database -d qdata -o /opt/oracle/products/11.2.0
srvctl add instance -d qdata -n rac1 -i qdata1
srvctl add instance -d qdata -n rac2 -i qdata2
/opt/oracle/products/11.2.0/bin/srvctl enable database -d qdata
/opt/oracle/products/11.2.0/bin/srvctl start database -d qdata
```

- 检查rac的网络

```
runcluvfy.sh comp nodecon -i <network-interface> -n <racnode1>,<racnode2>,<racnode3> -verbose
./runcluvfy.sh stage -pre crsinst -n rac1,rac2 -fixup -verbose
```

- 添加cluster nodes

```
$ORACLE_HOME/oui/bin/addNode.sh -silent "CLUSTER_NEW_NODES={qdata03}"

/opt/grid/products/11.2.0/oui/bin/runInstaller -addNode -invPtrLoc /opt/grid/products/11.2.0/oraInst.loc ORACLE_HOME=/opt/grid/products/11.2.0 -silent CLUSTER_NEW_NODES={qdata03}
```


How to Modify Private Network Information in Oracle Clusterware [ID 283684.1]


$OHOME/oraInst.loc

--按照单库模式来启动
在10g RAC以后，必须要先启动crs，才可以启动rac数据库。
但是如果确实十分紧急，可以rebuild oracle软件：
$ cd $ORACLE_HOME/rdbms/lib
$ make -f ins_rdbms.mk rac_off
$ cd $ORACLE_HOME/bin
$ relink oracle

这样rac_off了以后，再修改init.ora中的相应参数，比如cluster_database等，就可以以单实例的方式启动数据库实例了。




--ASM能mount，启动crs，或者oracle时出现错误
ORA-27140: attach to post/wait facility failed
ORA-27300: OS system dependent operation:invalid_egid failed with status: 1
ORA-27301: OS failure message: Operation not permitted
ORA-27302: failure occurred at: skgpwinit6
ORA-27303: additional information: startup egid = 501 (oinstall), current egid = 502 (dba)

解决方法：
chmod 6751 $GRID_HOME/bin/oracle

--11G endpoint listener 管理
11G RAC2默认采用grid gi来管理listener
srvctl stop listener -l LISTENER
srvctl start listener -l LISTENER

配置文件参考
/opt/grid/products/11.2.0/network/admin/endpoints_listener.ora

如果oracle服务在grid的listener上注册不了，可以参考：
chown oracle.asmadmin /opt/oracle/products/11.2.0/bin/oracle
chmod 6751 /opt/oracle/products/11.2.0/bin/oracle

#### 独占模式启动CRS

```
crsctl start crs -excl -nocrs
```


#### srvctl命令相关

```
instance,ASM,database,listener资源。
srvctl add database -d orcl -o /opt/oracle/products/11.2.0
srvctl add asm -n rac2 -i +ASM2 -o /opt/grid/products/11.2.0
srvctl add asm -n rac1 -i +ASM1 -o /opt/grid/products/11.2.0
srvctl add instance -d orcl -i orcl1 -n rac1
srvctl add instance -d orcl -i orcl2 -n rac2
srvctl add listener -l LISTENER -o /opt/grid/products/11.2.0
srvctl start listener -n qdata-rac1
srvctl add listener -n rac1 -o /opt/oracle/products/11.2.0 -l LISTENER_rac1
srvctl add listener -n rac2 -o /opt/oracle/products/11.2.0 -l LISTENER_rac2
srvctl add service -d irraca -s elearn -r irraca1,irraca2,irraca3 -P BASIC
srvctl start service -d irraca -s elearn

检验资源配置能否正常启动方法：
srvctl start listener -n rac1
srvctl start asm -n rac1
srvctl start instance -d orcl -i orcl1
上述命令成功的话，就可以使用crsctl 命令启停数据库资源了
crs_stop -all
crs_start -all
```

查看资源：

```
crsctl stat res -t
crsctl stat res -t -init
```

- 修改scan IP或scan name

```
# srvctl config scan
SCAN name: xff-scan, Network: 1/192.168.137.0/255.255.255.0/eth0
SCAN VIP name: scan1, IP: /xff-scan/192.168.137.245

# srvctl stop scan_listener
# srvctl stop scan

# srvctl modify scan -n xff-scan
# srvctl config scan
SCAN name: xff-scan, Network: 1/192.168.137.0/255.255.255.0/eth0
SCAN VIP name: scan1, IP: /xff-scan/192.168.137.248

# srvctl start scan
# srvctl start scan_listener

# crsctl status res -t

$ lsnrctl status LISTENER_SCAN1
修改数据库实例remote_listener参数
```

- Relocate a SCAN listener

```
oracle@test01 /export/home/oracle $ srvctl status scan
SCAN VIP scan1 is enabled
SCAN VIP scan1 is running on node test02
SCAN VIP scan2 is enabled
SCAN VIP scan2 is running on node test02
SCAN VIP scan3 is enabled
SCAN VIP scan3 is running on node test02
oracle@test01 /export/home/oracle $ srvctl relocate scan -i 1 -n test01
oracle@test01 /export/home/oracle $ srvctl status scan
SCAN VIP scan1 is enabled
SCAN VIP scan1 is running on node test01
SCAN VIP scan2 is enabled
SCAN VIP scan2 is running on node test02
SCAN VIP scan3 is enabled
SCAN VIP scan3 is running on node test02
```


- 修改RAC监听端口

```
srvctl modify scan_listener -p 1523
srvctl modify listener -l LISTENER -p 1523

srvctl stop scan_listener
srvctl start scan_listener
srvctl stop listener
srvctl start listener

--修改本地监听及SCAN监听端口
以oracle用户登陆，执行以下修改：
alter system set local_listener='(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=xx.xx.xx.xx)(PORT=1523))))' scope=both sid='db1';
alter system set local_listener='(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=xx.xx.xx.xx)(PORT=1523))))' scope=both sid='db2';
alter system set local_listener='(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=xx.xx.xx.xx)(PORT=1523))))' scope=both sid='db3';
alter system set remote_listener='gisscan:1523' scope=both sid='db1';
alter system set remote_listener='gisscan:1523' scope=both sid='db2';
alter system set remote_listener='gisscan:1523' scope=both sid='db3';

以grid用户登陆，执行以下修改：
alter system set local_listener='(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=xx.xx.xx.xx)(PORT=1523))))' scope=both sid='+ASM1';
alter system set local_listener='(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=xx.xx.xx.xx)(PORT=1523))))' scope=both sid='+ASM2';
alter system set local_listener='(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=xx.xx.xx.xx)(PORT=1523))))' scope=both sid='+ASM3';
alter system set remote_listener='gisscan:1523' scope=both sid='+ASM1';
alter system set remote_listener='gisscan:1523' scope=both sid='+ASM2';
alter system set remote_listener='gisscan:1523' scope=both sid='+ASM3';
```
