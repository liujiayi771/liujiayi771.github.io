---
title: Mac上制作Linux的U盘启动盘
date: 2018-10-31 10:05:33
tags: 
---
在windows上制作Linux的U盘启动盘非常容易，有很多工具，比如软碟通UltralISO、deepin深度启动盘制作工具等，只要正确使用这些工具就能很容易地制作一个U盘启动盘。

换了Mac电脑后，有时候要给服务器装Linux操作系统，还是找一台windows电脑来制作U盘启动盘，十分不方便。其实用Mac也能很容易地制作U盘启动盘，并且不需要借助什么工具，用dd命令就能完成U盘启动盘的制作。

#### 步骤
1. 插入U盘，使用diskutil指令来查看U盘的挂载点

	```bash
	$ diskutil list
	```
	系统输出类似如下的内容

	```bash
	# joey @ joey-Mac in ~ [9:45:27]
	$ diskutil list
	/dev/disk0 (internal):
	   #:                       TYPE NAME                    SIZE       IDENTIFIER
	   0:      GUID_partition_scheme                         24.0 GB    disk0
	   1:                        EFI EFI                     314.6 MB   disk0s1
	   2:                 Apple_APFS                         23.6 GB    disk0s2
	
	/dev/disk1 (internal, physical):
	   #:                       TYPE NAME                    SIZE       IDENTIFIER
	   0:      GUID_partition_scheme                        *1.0 TB     disk1
	   1:                        EFI EFI                     209.7 MB   disk1s1
	   2:                 Apple_APFS Container disk2         900.7 GB   disk1s2
	   3:       Microsoft Basic Data BOOTCAMP                99.3 GB    disk1s3
	
	/dev/disk2 (synthesized):
	   #:                       TYPE NAME                    SIZE       IDENTIFIER
	   0:      APFS Container Scheme -                      +900.7 GB   disk2
	                                 Physical Store disk1s2
	   1:                APFS Volume CoreStorage Fusion      576.8 GB   disk2s1
	   2:                APFS Volume Preboot                 44.0 MB    disk2s2
	   3:                APFS Volume Recovery                512.4 MB   disk2s3
	   4:                APFS Volume VM                      6.4 GB     disk2s4
	
	/dev/disk3 (external, physical):
	   #:                       TYPE NAME                    SIZE       IDENTIFIER
	   0:     FDisk_partition_scheme                        *15.7 GB    disk3
	   1:               Windows_NTFS joey                    15.7 GB    disk3s1
	```
	可以看到，我的U盘大小是16GB，对应上面输出的挂载点应该是位于`/dev/disk3`

2. 在写入系统镜像前，首先需要umount这个U盘，使用如下指令

	```bash
	$ diskutil unmountDisk /dev/disk3
	```
	会输出如下内容表示正确unmount
	
	```bash
	# joey @ joey-Mac in ~ [9:45:34]
	$ diskutil unmountDisk /dev/disk3
	Unmount of all volumes on disk3 was successful
	```
	此时，Mac的Finder中已经看不到之前挂载上的U盘了，但是使用`diskutil list`命令还是能够看到U盘的信息

3. 使用dd指令写入Linux系统镜像到U盘 

	```bash
	$ sudo dd if=/Users/joey/Downloads/CentOS-7-x86_64-DVD-1804.iso of=/dev/disk3 bs=1M
	```
	这里有一点需要注意，有些教程可能写的`bs=1m`，这里如果你的电脑装了GNU Coreutils，就是那个让`ls`能够彩色化输出的软件，这里用1m就会报错，报错内容为`dd: invalid number: ‘1m’`，这里把m大写就能很容易解决这个问题，但是究竟是什么原因造成1m不行的我也不是很清楚，只知道一个解决的办法。
	
	之后会输出如下，表示写入完成，大概要等几分钟
	
	```bash
	# joey @ joey-Mac in ~ [9:49:37] C:1
	$ sudo dd if=/Users/joey/Downloads/CentOS-7-x86_64-DVD-1804.iso of=/dev/disk3 bs=1M
	Password:
	4263+0 records in
	4263+0 records out
	4470079488 bytes (4.5 GB, 4.2 GiB) copied, 446.25 s, 10.0 MB/s
	```
	经过如上几个步骤，用Mac制作的U盘启动盘就制作完成了。
	
#### 一些问题
U盘制作启动盘之后U盘的可能无法被Mac系统读取了，会报一个错误，可见空间也会变的很小，这时候可以重新格式化U盘来进行恢复