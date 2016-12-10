
#### 查看RAM大小

```
MegaCli -cfgdsply -aALL   | grep "Memory"
```


#### Preserved Cache

```
MegaCli -GetPreservedCacheList -aN|-a0,1,2|-aALL
MegaCli -DiscardPreservedCache -Lx|-L0,1,2|-Lall -aN|-a0,1,2|-aALL
```
