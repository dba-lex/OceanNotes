
- 创建ADVM volume

```
SQL> alter diskgroup fiodg set attribute 'compatible.advm'='11.2';

Diskgroup altered.

SQL> ALTER DISKGROUP fiodg ADD VOLUME volume1 SIZE 10G;

Diskgroup altered.

SQL> ALTER DISKGROUP fiodg ENABLE VOLUME volume1;

Diskgroup altered.

SQL> select VOLUME_DEVICE from V$ASM_VOLUME where VOLUME_NAME='VOLUME1';

VOLUME_DEVICE
--------------------------------------------------------------------------------
/dev/asm/volume1-467
```

- 查看设备

```
grid@rac2:/home/grid>fdisk -l /dev/asm/volume1-467

Disk /dev/asm/volume1-467: 10.7 GB, 10737418240 bytes
255 heads, 63 sectors/track, 1305 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

Disk /dev/asm/volume1-467 doesn't contain a valid partition table
```

- 创建ACFS集群文件系统

```
# mkdir /acfs

[root@rac2 /opt/grid/products/11.2.0]
#/sbin/mkfs -t acfs -n ACFSVOL1 /dev/asm/volume1-467
mkfs.acfs: version                   = 11.2.0.4.0
mkfs.acfs: on-disk version           = 39.0
mkfs.acfs: volume                    = /dev/asm/volume1-467
mkfs.acfs: volume size               = 10737418240
mkfs.acfs: Format complete.
```

- mount文件系统，注册挂载点

```
[root@rac2 /opt/grid/products/11.2.0]
#/sbin/acfsutil registry -a -f /dev/asm/volume1-467 /acfs/
acfsutil registry: mount point /acfs successfully added to Oracle Registry
```

- 在线修改ACFS文件系统大小

```
#/sbin/acfsutil size 15G /acfs
acfsutil size: new file system size: 16106127360 (15360MB)
```

- 查看volume信息：

```
#/sbin/advmutil volinfo /dev/asm/volume1-467
Device: /dev/asm/volume1-467
Interface Version: 1
Size (MB): 15360
Resize Increment (MB): 32
Redundancy: unprotected
Stripe Columns: 4
Stripe Width (KB): 128
Disk Group: FIODG
Volume: VOLUME1
Compatible.advm: 11.2.0.0.0
```
