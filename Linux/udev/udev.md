


```
cd /dev/mapper
for i in ls *; do
printf "%s %s\n" "$i" "$(udevadm info --query=all --name=/dev/mapper/$i |grep -i dm_uuid)"; done
```



```
udevadm info --query=all --path=/sys/block/sdaa

/sbin/scsi_id --whitelisted --replace-whitespace --device=/dev/emcpowera

fdisk -l | grep emcpower | grep "107.4 GB"
fdisk -l | grep emcpower | grep "214.8 GB"


KERNEL=="emcpower*", SUBSYSTEM=="block", PROGRAM=="/sbin/scsi_id --whitelisted --replace-whitespace --device=/dev/$name", RESULT=="360000970000498700082533030343746", NAME="asmdisk/asm-arch-d", OWNER="grid", GROUP="asmadmin", MODE="0660"


> /etc/udev/rules.d/96-asmdisk.rules

--107G盘
fdisk -l | grep emcpower | grep "107.4 GB" > /tmp/fdisk.txt
for i in `awk -F':' '{print $1}' /tmp/fdisk.txt |awk '{print $2}'`; do
     scsiid=`/sbin/scsi_id --whitelisted --replace-whitespace --device=${i}`
     diskname=`expr substr ${i} 14 3`
     echo "KERNEL==\"emcpower*\", SUBSYSTEM==\"block\", PROGRAM==\"/sbin/scsi_id --whitelisted --replace-whitespace --device=/dev/\$name\", RESULT==\"$scsiid\", NAME=\"asmdisk/asm-data-${diskname}\", OWNER=\"grid\", GROUP=\"asmadmin\", MODE=\"0660\"" >> /etc/udev/rules.d/96-asmdisk.rules
done

--214G盘
fdisk -l | grep emcpower | grep "214.8 GB" > /tmp/fdisk.txt
for ii in `awk -F':' '{print $1}' /tmp/fdisk.txt |awk '{print $2}'`; do
     scsiiid=`/sbin/scsi_id --whitelisted --replace-whitespace --device=${ii}`
     disknamee=`expr substr ${ii} 14 3`
     echo "KERNEL==\"emcpower*\", SUBSYSTEM==\"block\", PROGRAM==\"/sbin/scsi_id --whitelisted --replace-whitespace --device=/dev/\$name\", RESULT==\"$scsiiid\", NAME=\"asmdisk/asm-arch-${disknamee}\", OWNER=\"grid\", GROUP=\"asmadmin\", MODE=\"0660\"" >> /etc/udev/rules.d/96-asmdisk.rules
done
```
