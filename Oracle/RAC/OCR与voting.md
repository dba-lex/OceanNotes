- 查看ocr地址

```
ocrcheck
或者
cat /etc/oracle/ocr.loc
```

- 查看voting

```
crsctl query css votedisk
```

- 新增voting盘

```
crsctl add css votedisk path_to_voting_disk
```

- 删除voting盘

```
crsctl delete css votedisk  /dev/titan/iqn.titan.opensource.target10_192.168.0.31-3260_sde_tsas-sdd
```

- 添加ocr备份路径（确保每个节点都有这个目录）

```
/opt/grid/products/11.2.0/bin/ocrconfig   -backuploc /data/asmocr_back/
```

- 手工备份ocr

```
/opt/grid/products/11.2.0/bin/ocrconfig  -manualbackup
```

- 恢复ocr

```
/opt/grid/products/11.2.0/bin/ocrconfig -restore /opt/grid/products/11.2.0/cdata/rac-redhat/week.ocr
```

- 修改voting盘路径（注意，不好用，总是出错，不如添加新的删除老的）

```
方法1
crsctl replace votedisk /dev/titan/iqn.titan.opensource.target10_192.168.0.31-3260_sdd_tsas

Now formatting voting disk: /dev/titan/iqn.titan.opensource.target10_192.168.0.31-3260_sdd_tsas-sdc.
CRS-4256: Updating the profile
Successful addition of voting disk 138c32d67db84f7fbf2c78eca010a47e.
Successful deletion of voting disk 0e1e349cd5314ffabfa801d308504aff.
CRS-4256: Updating the profile
CRS-4266: Voting file(s) successfully replaced

方法2
crsctl add css votedisk /dev/raw/raw1 -purge
```

How to re-install CRS when having RDBMS oracle home        
文档 ID:   456021.1

- Oracle 11GR2 RAC reconfig GI DOC 942166.1

```
留一个节点，在其他节点运行
/opt/grid/products/11.2.0/crs/install/rootcrs.pl -verbose -deconfig -force

在最后一个节点，运行（这个会清除OCR,VOTING和他们的ASM DG）
/opt/grid/products/11.2.0/crs/install/rootcrs.pl -verbose -deconfig -force -lastnode

逐个节点运行root.sh(和安装clusterware一样)
/opt/grid/products/11.2.0/root.sh

注意：有时候在运行reconfig时可能会hang住，可以通过几种方式解决
1.disable crs，重启主机，让系统CRS不再启动
2.使用/opt/grid/products/11.2.0/crs/install/rootcrs.pl -verbose -deconfig -force
的方式来做，手工去dd votingdg
```

- NFS OCR&voting

Doc ID 359515.1
