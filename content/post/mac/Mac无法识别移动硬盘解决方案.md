+++
title = 'Mac无法识别移动硬盘解决方案.md'
date = 2025-09-05T22:34:05+08:00
draft = true
categories = [ "Mac" ]
+++

## 背景

我升级完Mac系统之后，插入移动硬盘，发现无法识别：

![alt text](image-2.png)

原本是 2T 的硬盘现在只显示 4G，明显出现了问题。

## 解决

1、先找到外接移动硬盘，在终端执行下面命令：
```shell
➜  ~ diskutil list
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *1.0 TB     disk0
   1:             Apple_APFS_ISC Container disk1         524.3 MB   disk0s1
   2:                 Apple_APFS Container disk3         994.7 GB   disk0s2
   3:        Apple_APFS_Recovery Container disk2         5.4 GB     disk0s3

/dev/disk3 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +994.7 GB   disk3
                                 Physical Store disk0s2
   1:                APFS Volume Macintosh HD            11.6 GB    disk3s1
   2:              APFS Snapshot com.apple.os.update-... 11.6 GB    disk3s1s1
   3:                APFS Volume Preboot                 14.1 GB    disk3s2
   4:                APFS Volume Recovery                2.1 GB     disk3s3
   5:                APFS Volume Data                    846.3 GB   disk3s5
   6:                APFS Volume VM                      20.5 KB    disk3s6

/dev/disk6 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *2.0 TB     disk6
   1:                        EFI EFI                     209.7 MB   disk6s1
   2:                  Apple_HFS                         2.0 TB     disk6s2
                    (free space)                         135.2 MB   -
   3:       Microsoft Basic Data G-UTILITIES             4.1 GB     disk6s3

```

从终端内容看，外接移动硬盘名称是 `disk6`，容量是 2TB，硬盘的格式没有明确标注，从 Microsoft Basic Data 几个字样推断，再加上之前在 Windows 系统也能正常使用，应该能兼容 Windows 和 Mac 的 exFAT 格式。

disk6 下面包括两个盘符标记：disk6s1、 disk6s2、disk6s3，多个盘符表示硬盘包含了多个分区，可以看到 disk6s1 的 TYPE NAME 的 EFI。

EFI 分区在我们 Mac 系统这边没有什么意义，我们实际存储数据的硬盘分区是 disk6s2，后面的命令操作对哦想和执行结果也是针对硬盘分区 disk6s2 来讲的

2、加载移动硬盘，在终端执行下面命令

```shell

# mount 命令可以只挂在硬盘的某个分区
diskutil mount /dev/disk6s2

# mountdisk 命令可以挂在整个硬盘，包括所有分区
diskutil mountdisk /dev/disk6
```