# EMMC相关操作：

## 查看mmc的内容：

切换当前mmc：

```shell
mmc dev 0
```

查看mmc内容

```
ls mmc 0:1
```

## 下载mmc里面的文件：

```shell
load mmc 0:1 0x10000000 /image.ub
```

说明：把emmc0的第一个分区里面的imag.ub下载到0x1000 0000地址上

## 存储文件到EMMC：

```shell
save mmc 0:1 ${loadaddr} system.bit ${filesize}
```

说明：${loadaddr} 这个地址需要修改为文件的地址；${filesize}这个他会自动默认是刚刚下载的文件大小，不用修改









# FLASH相关操作：

## 擦除FLASH：

```shell
## 查询FLASH设备：
sf probe
SF: Detected n25q512a with page size 256 Bytes, erase size 64 KiB, total 64 MiB
```

```shell
## 擦除FLASH
sf erase 0 0x40 00000 
```

 说明：此时的FLASH大小为64M，五个0以上代表M，0x40 = 64

## 写入FLASH

```shell
## 擦除FLASH
sf erase 0 0x40 00000 

## 写入FLASH （前提：地址下有文件）
sf write 0x10000000 0 ${filesize}
```

说明：起始偏移量：0 ,从0地址开始写





# USB相关操作：

启动/重启USB：

```
usb start 
usb reset
```

查询USB文件：

```
ls usb 0:0

System Volume Information/
     2710   boot.scr
 46696249   rootfs.tar.gz
  9417496   image.ub
    38730   system.dtb
  6625641   zynqmp_wrapper.bit
  6625641   system.bit

6 file(s), 1 dir(s)

```

## 下载USB - u盘文件：

```shell
load usb 0 0x10000000 system.bit
```









# 修改zynq启动项

## 从emmc启动内核

```shell
## 添加指令
setenv my_emmc_boot "mmc dev 0:1 && load mmc 0:1 0x10000000 /image.ub && bootm 0x10000000"

## 将bootcmd的命令修改成my_emmc_boot
setenv bootcmd "run my_emmc_boot" 

## 保存环境变量
saveenv
```

## 从emmc启动bit

```shell
## 添加指令
setenv my_emmc_bit "mmc dev 0:1 && load mmc 0:1 0x10000000 system.bit && fpga loadb 0 0x10000000 ${filesize}"
```

说明：loadb是加载的.bit，如果改为load 就代表加载.bin文件





# 指令查询和添加：

```shell
## 查询启动内核指令：
print bootcmd

```



