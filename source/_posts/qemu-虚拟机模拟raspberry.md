---
title: qemu 虚拟机模拟raspberry
date: 2017-04-17 20:41:18
tags: raspberry
---

## 1，下载树莓派
到此目录下下载树莓派的镜像文件
```
    https://downloads.raspberrypi.org/raspbian/
```
## 2，安装qemu
```
apt-get install qemu binfmt-support qemu-user-static
```
## 3，下载树莓派启动内核
```
https://github.com/dhruvvyas90/qemu-rpi-kernel
```
<!-- more -->
## 4，修改系统配置
### 4.1 将镜像文件映射成loop设备文件
```
mkdir rootfs
sudo kpartx -av 2016-09-23-raspbian-jessie.img
sudo mount /dev/mapper/loop0p2 rootfs
cd rootfs
```

### 4.1 将etc/fstab文件中的mmcblk的挂载注释掉
```
proc            /proc           proc    defaults          0       0
#/dev/mmcblk0p1  /boot           vfat    defaults          0       2
#/dev/mmcblk0p2  /               ext4    defaults,noatime  0       1
# a swapfile is not a swap partition, no line here
#   use  dphys-swapfile swap[on|off]  for that
```
### 4.2 修改/etc/ld.so.preload文件注释掉引用的动态库
```
#/usr/lib/arm-linux-gnueabihf/libarmmem.so
```
### 4.3 umount掉rootfs,并移除loop设备
```
sudo umount /mnt/img1
sudo kpartx -d your-image.img
```
### 4.4 启动qemu
```
qemu-system-arm -kernel ./kernel-qemu-4.4.21-jessie -cpu arm1176 -m 256M -M versatilepb -serial stdio -append "root=/dev/sda2 console=ttyAMA0 panic=1 rootfstype=ext4 rw" -drive "file=2016-09-23-raspbian-jessie.img,index=0,media=disk,format=raw" -redir tcp:2222::22

qemu-system-arm -kernel ./kernel-qemu-4.4.21-jessie -cpu arm1176 -m 256M -M versatilepb -serial stdio -append "root=/dev/sda2 console=ttyAMA0 panic=1 rootfstype=ext4 rw" -drive "file=2016-09-23-raspbian-jessie.img,index=0,media=disk,format=raw"  -net nic -net tap,ifname=tap0,script=no
```
## 5扩展分区
直接启动发现系统所剩空间不多
```
df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root       3.9G  3.6G   94M  98% /
devtmpfs        124M     0  124M   0% /dev
tmpfs           124M     0  124M   0% /dev/shm
tmpfs           124M  4.5M  120M   4% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           124M     0  124M   0% /sys/fs/cgroup
tmpfs            25M     0   25M   0% /run/user/1000
```
因此需要扩展分区
### 5.1 扩展镜像分区
qemu-img resize 2016-09-23-raspbian-jessie.img +5G
```
sudo fdisk 2016-09-23-raspbian-jessie.img 

Command (m for help): p

Disk 2016-09-23-raspbian-jessie.img: 9717 MB, 9717153792 bytes
255 heads, 63 sectors/track, 1181 cylinders, total 18978816 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xd94ffdb3

                         Device Boot      Start         End      Blocks   Id  System
2016-09-23-raspbian-jessie.img1            8192      137215       64512    c  W95 FAT32 (LBA)
2016-09-23-raspbian-jessie.img2          137216     8493055     4177920   83  Linux

Command (m for help): p

Disk 2016-09-23-raspbian-jessie.img: 9717 MB, 9717153792 bytes
255 heads, 63 sectors/track, 1181 cylinders, total 18978816 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xd94ffdb3

                         Device Boot      Start         End      Blocks   Id  System
2016-09-23-raspbian-jessie.img1            8192      137215       64512    c  W95 FAT32 (LBA)
2016-09-23-raspbian-jessie.img2          137216     8493055     4177920   83  Linux

Command (m for help): d
Partition number (1-4): 2

Command (m for help): n
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): p
Partition number (1-4, default 2): 
Using default value 2
First sector (2048-18978815, default 2048): 137216
Last sector, +sectors or +size{K,M,G} (137216-18978815, default 18978815): 
Using default value 18978815

Command (m for help): w
The partition table has been altered!

Syncing disks.

```
查看分区结果
```
fdisk -l 2016-09-23-raspbian-jessie.img 

Disk 2016-09-23-raspbian-jessie.img: 9717 MB, 9717153792 bytes
255 heads, 63 sectors/track, 1181 cylinders, total 18978816 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xd94ffdb3

                         Device Boot      Start         End      Blocks   Id  System
2016-09-23-raspbian-jessie.img1            8192      137215       64512    c  W95 FAT32 (LBA)
2016-09-23-raspbian-jessie.img2          137216    18978815     9420800   83  Linux
```
再次启动查看，磁盘空间改变
```
df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root       8.8G  3.6G  4.9G  43% /
devtmpfs        124M     0  124M   0% /dev
tmpfs           124M   68K  124M   1% /dev/shm
tmpfs           124M  4.5M  120M   4% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           124M     0  124M   0% /sys/fs/cgroup
tmpfs            25M  4.0K   25M   1% /run/user/1000
```

## 6 增加内存
由于只有使用256M内存，所以系统反应有些慢，因此可以增加swap来提高性能
修改/etc/dphys-swapfile文件
```
#CONF_SWAPSIZE=100
CONF_SWAPSIZE=1024
```
重启服务
```
sudo /etc/init.d/dphys-swapfile stop
sudo /etc/init.d/dphys-swapfile start
```
查看
```
free -m
             total       used       free     shared    buffers     cached
Mem:           247        222         25          5         10        156
-/+ buffers/cache:         55        192
Swap:         1023          0       1023
```
## 7 修改设备名称
在虚拟机中磁盘别识别位sda，而在物理系统中则位mmcblk0*，因此为了保持一直，修改udev中的连接
```
/etc/udev/rules.d/90-qemu.rules 
KERNEL=="sda", SYMLINK+="mmcblk0"
KERNEL=="sda1", SYMLINK+="mmcblk0p1"
KERNEL=="sda2", SYMLINK+="mmcblk0p2"
```

## 8超频
```
sudo raspi-config
```
选择8 over clock，在选择Turbo模式
