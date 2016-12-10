



- 查看命令的具体用法，本例为lsct

```
ASMCMD> help lsct
```

- find命令

```
find --type datafile +ocrvote sys*


SQL> select distinct type from v$asm_file;

TYPE
--------------------------------------------------------------------------------
ARCHIVELOG
ASMPARAMETERFILE
CONTROLFILE
DATAFILE
OCRFILE
ONLINELOG
PARAMETERFILE
TEMPFILE
```

- 查看磁盘组信息

```
asmcmd lsdg
asmcmd lsdg --discovery
```

- cp命令

```
cp:不仅可以在ASM和OS之间复制文件，也可以在不同的ASM Instance和Diskgroup之间复制文件;
cp +dgtest/test/datafile/USERS.264.646186565 users.dbf
cp +dgtest/test/datafile/USERS.264.646186565 /home/grid/users.dbf
```

- 将disk group中的metadata备份到文件

```
md_backup /tmp/backupfile -G DATAGP
```

- 将备份文件中的metadata恢复到disk group

```
md_restore -full -G data --silent /tmp/file
```

- 列出disk group的属性

```
lsattr -l -G datadg
```

- 设置disk group的属性

```
setattr -G DATAGP compataible.asm 11.2.0.0.0
```

- 显示local clients的open files;

```
lsof -G DATAGP
```

- check 或 repair disk group 的metadata;

```
chkdg --repair DATAGP
```

- drop disk group

```
dropdg -r -f DATAGP
```

- 查看I/O statics通过v$asm_disk_iostat

```
iostat -G DATAGP
```

- list ASM disks

```
Displays the TOTAL_MB, FREE_MB, OS_MB,NAME, FAILGROUP,LIBRARY, LABEL, UDID, PRODUCT, REDUNDANCY, and PATH columns of the V$ASM_DISK view
lsdsk -k

Displays the GROUP_NUMBER, DISK_NUMBER, INCARNATION,MOUNT_STATUS, HEADER_STATUS, MODE_STATUS, STATE,and the PATH columns of the V$ASM_DISK view
lsdsk -p -G DATAGP /dev/raw/*

Restricts results to only disks having membership status equal to CANDIDATE
lsdsk --candidate

Displays the disks that are visible to some but not all active instances. These are disks that, if included in a disk group, will cause the mount of that disk group to fail on the instances where the disks are not visible.
lsdsk -M

Displays the CREATE_DATE, MOUNT_DATE, REPAIR_TIMER, and the PATH columns of the V$ASM_DISK view
lsdsk -t
```

- lsod: list open ASM disks

```
lsod -G DATAGP
```

- mkdg: create disk group based on a xml file;

```
mkdg DATAGP_config.xml
```

- mount: mount a disk group;

```
mount -f DATAGP;
mount --restrict DATAGP;
mount -a
```

- offline: offline disks or failure groups that belong to disk group.

```
offline -G DATAGP -F FG1
```

- online: online disks or a failure group

```
online -G DATAGP -D data_0001 --power=3
```

- rebal: rebalance a disk group;

```
rebal --power 4 DATAGP
```

- remap: mark blocks as unusable on the disk and relocates data;

```
remap DATAGP data_0001 500-700
```

- umount: dismount a disk group;

```
unmount -f DATAGP
```

- pwcopy: copy password file;

```
pwcopy --asm +DG/mydir/mypwfile +DG1/mypwfile
```

- pwcreate: create password file for sys;

```
pwcreate --asm +DG/mdir/mypwfile 'welcome'
```

- pwdelete: delete password file;

```
pwdelete --asm +DG/mydir/mypwfile
```

- pwget: get the location of password file;

```
pwget --asm
```

- pwmove: move password file;

```
pwmove --asm +DG/mydir/mypwfile +DG1/mypwfile
```

- pwset: set the location of password file;

```
pwset -dbuniquename aime1 +DG/mydir/mypwfile
```

- dsget: get the discovery disk string;

```
dsget
```

- dsset: set the discovery disk string;

```
dsset /dev/raw/*
```

- lsop: list current operations on disk group from v$asm_operation;

```
lsop
```

- shutdown: shutdown ASM instance;

```
shut immediate
```

- spbackup: backup ASM Spfile;

```
spbackup +DATA/asm/asmprameterfile/register.323.234 +DATA/spf.bak
```

- spcopy: copy spfile;

```
spcopy +DATA/asm/asmprameterfile/register.323.234 +DATA/spf.ora
```

- spget: get the spfile location;

```
spget
```

- spmove: move spfile;

```
spmove +DATA/spf.ora +DATA1/spf.ora
```

- spset: set the location of spfile;

```
spset +DATA/spf.ora
```

- startup: start up ASM instance;

```
startup --nomount --pfile asm.ora
```

- chtmpl:改变template的属性；

```
chtmpl -G DATAGP --redundancy high --striping fine mytemplate
```

- lstmpl: list templates;

```
lstmpl -l -G DATAGP
```

- mktmpl: add template to disk group;

```
mktmpl -G DATA --redundancy mirror --striping coarse
```

- rmtmpl: remove template from disk group;

```
rmtmpl -G DATAGP mytp
```

- chgrp: change user group of files;  

```
chgrp asm-data +data/mydir/a.f
```

- chmod: change permissions of files;

```
chmod 640 a.f
```

- chown: change owner of files;

```
chown user:usergroup a.f
```

- groups: list all user groups of a user;

```
groups DATAGP user
```

- grpmod: add or remove OS users to ASM user group;

```
grpmod --add fra asm_fra oracle1 oracle2
```

- lsgrp: list all ASM user groups;

```
lsgrp -a
```

- lspwusr: list users from ASM password file;

```
lspwusr
```

- lsusr: list users in a disk group;

```
lsusr -G DATAGP
```

- mkgrp: create new ASM user group;

```
mkgrp DATAGP asm_data oracle1 oracle2
```

- mkusr: add OS user to a disk group;

```
mkusr DATA oracle1
```

- orapwusr: add, drop, modify ASM password file user;  

```
orapwusr --add --privilege sysdba hrusr
```

- passwd: change password of a user;

```
passwd oracle2
```

- rmgrp: remove a user group from disk group;

```
rmgrp DATAGP asm_data
```

- rmusr: remove a OS user from disk group;

```
rmusr DATAGP oracle2
```

- rpusr: replace OS user1 with OS user2;

```
rpusr DATAGP oracle1 oracle2
```

- volcreate: create an ADVM volume in disk group;

```
volcreate -G DATA -s 10G --width 64K --column 8 volume1
```

- voldelete: delete an ADVM volume;

```
voldelete -G DATAGP volume1
```

- voldisable: disable an ADVM volumes in mounted disk groups and remove the volume device on the local node;

```
voldisable -G DATAGP volume1
```

- volenable: enable ADVM volume in mounted disk groups;

```
volenable -G DATAGP volume1
```

- volinfo: display information of ADVM volumes;

```
volinfo -G DATAGP volume1
```

- volresize: resize an ADVM volume;

```
volresize -G DATAGP -s 20G volume1
```

- volset: set attributes of ADVM volume;

```
volset -G DATA --usagestring 'no file system attached'  volume1
```

- volstat: report I/O statistics of ADVM volume;

```
volstat -G DATAGP
```
