
- 查看HCA卡端口的物理和逻辑端口状态、HCA卡firmware版本

```
ibstat
```

- 查看HCA卡物理端口与OS网卡设备的对应关系

```
ibdev2netdev
```

- 查看当前ofed版本

```
ofed_info
```

- 查看当前网络中所有IB交换机

```
ibswitches
```

- 查看当前SM所在的主机

```
$ sminfo
sminfo: sm lid 37 sm guid 0x7cfe90030016aef1, activity count 4484381 priority 14 state 3 SMINFO_MASTER

$ smpquery ND -L 37
Node Description:.................b1nstcx02 HCA-1
```

- 安装ofed

```
mlnxofedinstall --force-fw-update --force
```

- 网络拓扑

```
iblinkinfo
```

- update firmware

方法一：

```
mlnxofedinstall --fw-update-only
```

方法二：

```
ibv_devinfo 获取board id
如：
board_id:			MT_1090120019

ibstat | grep CA  获取CA type
如：
CA type: MT4099

Download the Mellanox Firmware Tools (MFT):
http://www.mellanox.com/page/firmware_HCA_FW_update
cd mft-4.5.0-31-x86_64-rpm
./install.sh

mst start
Starting MST (Mellanox Software Tools) driver set
Loading MST PCI module - Success
Loading MST PCI configuration module - Success
Create devices


mst status
MST modules:
------------
    MST PCI module loaded
    MST PCI configuration module loaded

MST devices:
------------
/dev/mst/mt4099_pciconf0         - PCI configuration cycles access.
                                   domain:bus:dev.fn=0000:02:00.0 addr.reg=88 data.reg=92
                                   Chip revision is: 01
/dev/mst/mt4099_pciconf1         - PCI configuration cycles access.
                                   domain:bus:dev.fn=0000:81:00.0 addr.reg=88 data.reg=92
                                   Chip revision is: 01
/dev/mst/mt4099_pci_cr0          - PCI direct access.
                                   domain:bus:dev.fn=0000:02:00.0 bar=0xc7500000 size=0x100000
                                   Chip revision is: 01
/dev/mst/mt4099_pci_cr1          - PCI direct access.
                                   domain:bus:dev.fn=0000:81:00.0 bar=0xfbe00000 size=0x100000
                                   Chip revision is: 01


flint -d /dev/mst/mt4099_pciconf1 -i fw-ConnectX3-rel-2_40_5000-MCX354A-FCB_A2-A5-FlexBoot-3.4.746.bin burn

    Current FW version on flash:  2.35.5100
    New FW version:               2.40.5000

Burning FS2 FW image without signatures - OK
Restoring signature                     - OK
```
