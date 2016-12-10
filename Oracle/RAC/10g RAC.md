

#OCR分为普通ocr和镜像ocr，一般1是普通，2是镜像

#添加OCR镜像
su - root
./ocrconfig -replace ocrmirror /dev/raw/raw2

#删除OCR镜像
./ocrconfig -replace ocrmirror

#删除普通ocr(普通ocr删除后，镜像ocr就变成了普通ocr)
./ocrconfig -replace ocr

#10G的crs起停
#crs的启动和停止

```
crsctl stop crs

/etc/init.d/init.cssd start
/etc/init.d/init.crs start
```


raw /dev/raw/raw1 /dev/qdata/mpath-1qdata-idc-sz-qdata4-virtual-ocr
raw /dev/raw/raw2 /dev/qdata/mpath-1qdata-idc-sz-qdata4-virtual-voting
raw /dev/raw/raw3 /dev/qdata/mpath-1qdata-idc-sz-qdata5-virtual-ocr
raw /dev/raw/raw4 /dev/qdata/mpath-1qdata-idc-sz-qdata5-virtual-voting
raw /dev/raw/raw5 /dev/qdata/mpath-1qdata-idc-sz-qdata6-virtual-voting

chown -Rf oracle.dba /dev/raw/*
chown -Rf oracle.dba /dev/raw/*
chown -Rf oracle.dba /dev/raw/raw5
