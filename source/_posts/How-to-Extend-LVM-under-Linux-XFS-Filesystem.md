---
title: How to Extend LVM in Linux
date: 2017-11-06 15:16:07
tags:
---
<center> Linux xfs 檔案系統下增長 LVM 空間</center>
===
<br>

原本 f25 /dev/fedora/root 有 15G ，裝 cuda8.0 也要 13G 上下，只好來 enlarge

## Overview

能夠線上 enlarge/extend 有幾個條件：
a. 該 partition 是 LVM
b. PV、VG 還有空間，也就是該 LVM 的上一層和上上一層都有空間可以增加，如果不夠可能要先增加實體硬碟

如何 Mount LVM ，可以看之前的筆記[用 LVM 管理 CentOS 7 掛載的資料碟](https://wyde.github.io/2017/10/13/How-to-Mount-a-Second-Drive-Managed-by-LVM/#more)

關於 a 點可以用 `$ lsblk -l` 來確認， b 的話可以用 `$ pvs`、`$ vgs`、`$ lvs`來一步步確認空間是否足夠，幸運的是我做了 200G 的系統碟，只有用到 15G ，如果要 extend `/` 至 30G ，空間顯然還夠

步驟會是
1. extend LVM
2. extend file system

## Extend LVM

lvextend 前
```
    $ sudo lvs
    LV   VG     Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
    root fedora -wi-ao---- 15.00g  
```

`lvextend` 的參數
- `-l` 可接 PE 數量或剩餘空間的百分比
- `-L` 可接容量單位
```
    $ sudo lvextend -L+15G /dev/fedora/root
```

lvextend 後
```
    $ sudo lvs
    LV   VG     Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
    root fedora -wi-ao---- 30.00g                                                    
```

## extend file system

由於 f25+ 、 CentOS 7 已是 xfs file system， `resize2fs` 只對 ext 家族適用，這裡我們要用 `xfs_growfs` 指令，如果不加參數就是把剩下的(LVM)空間撐滿
```
    $ sudo xfs_growfs /dev/fedora/root 
    meta-data=/dev/mapper/fedora-root isize=512    agcount=4, agsize=983040 blks
             =                       sectsz=512   attr=2, projid32bit=1
             =                       crc=1        finobt=1 spinodes=0 rmapbt=0
             =                       reflink=0
    data     =                       bsize=4096   blocks=3932160, imaxpct=25
             =                       sunit=0      swidth=0 blks
    naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
    log      =internal               bsize=4096   blocks=2560, version=2
             =                       sectsz=512   sunit=0 blks, lazy-count=1
    realtime =none                   extsz=4096   blocks=0, rtextents=0
    data blocks changed from 3932160 to 7864320 
```

可以看到 data blocks 從 3932160 增加到 7864320，也可以用 `xfs_info` 指令查看 xfs 的狀況及大小

## Reference
- [Redhat - Growing Logical Volumes](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Logical_Volume_Manager_Administration/lv_extend.html)
- [Tecmint - How to Extend/Reduce LVM’s (Logical Volume Management) in Linux – Part II](https://www.tecmint.com/extend-and-reduce-lvms-in-linux/)
