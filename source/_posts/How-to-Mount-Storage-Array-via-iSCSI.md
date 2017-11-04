---
title: How to Mount Storage Array via iSCSI
date: 2017-09-29 22:37:15
tags:
- iSCSI
- ta217
- system administration
- 機房管理
---

# <center>透過 iSCSI 連結 disk array</center>
<br>

最近把兩台 Dell PowerEdge R610 透過 iSCSI(IP-SAN) 走 10GBase-T/RJ45 掛上 Dell PowerVault MD3800i，簡單地說就是 server 需要大容量的儲存空間，又不想要太貴的光纖介面，折衷一下。 iSCSI 不只可以用在 disk array 和 server 之間的連接，也不一定要 10G ，用一般 server 也可以挖一塊硬碟空間透過 ip 層來掛給另一台機器用。被掛的機器上要設定 iSCSI target ，掛的機器上要設定 iSCSI client，由於 MD3800i 的 iSCSI target 是透過 Dell 的 PowerVault Modular Disk Storage Manager(MDSM) 軟體來管理，所以這邊就不提 iSCSI target 怎麼 initialize ，以下教學假設 MD3800i 或 disk array 已做好 raid 、 切好 virtual disk 、 disk group

## 實驗環境

Prerequisite: 
- disk array 上的硬碟共 4TB * 12，4 顆作 raid 10 約 7T ，8 顆作 raid 10 ，約 14 T (自行依需求配置)
- 兩台 server 灌 CentOS 7
- 一台筆電裝好 MDSM 軟體

Resource: [PDF: Dell PowerVault MD3800i 部署指南](http://topics-cdn.dell.com/pdf/powervault-md3800i_deployment%20guide3_zh-cn.pdf)

Network:

![架構圖](https://i.imgur.com/Y5yEliJ.png)

| iSCSI Client | int | Client IP | Target IP | Cotroller | Port |
| :---: | :---: | :---: | :---: | :---: | :---: |
| server1 | p1p1 | 192.168.130.18 | 192.168.130.20 | 0 | 0 |
| server1 | p1p2 | 192.168.131.18 | 192.168.131.15 | 1 | 1 |
| server2 | p1p1 | 192.168.130.19 | 192.168.130.15 | 1 | 0 |
| server2 | p1p2 | 192.168.131.19 | 192.168.131.20 | 0 | 1 |

Goal: 依照架構圖，透過 iSCSI 各掛上一顆 Virtual Disk，這個架構的好處是每台 server 的 10G 網卡兩個 port 各接 disk array 的一個 controller ，可以避免單點失效，在 client 端就要設 multipath 告訴系統這兩張卡看到的 iSCSI storage 是同一顆


## 安裝步驟

安裝時會在 client 和 target 之間交錯，這裡以 server1 作例子， server2 請如法炮製，但是要換 ip

### client 端: server1
```
    $ sudo yum update
    $ sudo yum install epel-release iscsi-initiator-utils
    $ sudo iscsiadm -m discovery -t st -p 192.168.130.20
    $ sudo iscsiadm -m discovery -t st -p 192.168.131.15
```

檢查一下，把不屬於 server1 的 portal 砍掉
```
    $ sudo iscsiadm -m node
    $ sudo iscsiadm -m node -o delete -T iqn.1984-05.com.dell:powervault.md3800i.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx --portal 192.168.130.20:3260,1
    $ sudo iscsiadm -m node -o delete -T iqn.1984-05.com.dell:powervault.md3800i.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx --portal 192.168.130.15:3260,2
```

登入，如果需要登出，結尾 l->u
```
    $ sudo iscsiadm -m node -T iqn.1984-05.com.dell:powervault.md3800i.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx --portal 192.168.130.20:3260,1 -l
    $ sudo iscsiadm -m node -T iqn.1984-05.com.dell:powervault.md3800i.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx --portal 192.168.131.15:3260,2 -l
```

### target 端

- 設定 Host (略)
- 設定 Host Mapping (略)

### client 端

這時 `lsblk` 應該可以看到那顆 VD ，輸出長得像
```
    NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sda           8:0    0 279.4G  0 disk 
    ├─sda1        8:1    0     1G  0 part /boot
    └─sda2        8:2    0 278.3G  0 part 
      ├─cl-root 253:0    0    50G  0 lvm  /
      ├─cl-swap 253:1    0  23.6G  0 lvm  [SWAP]
      └─cl-home 253:2    0 204.6G  0 lvm  /home
    sdb           8:32   0  14.5T  0 disk 
    └─sdb1        8:33   0  14.5T  0 part 
    sdc           8:48   0  14.5T  0 disk 
    └─sdc1        8:49   0  14.5T  0 part 
    sr0          11:0    1  1024M  0 rom 
```

然後來裝 multipath (如果網卡不需要 failover ，可以跳過 multipath)
```
    $ sudo yum install -y device-mapper-multipath
    $ sudo vim /etc/multipath.conf
```

一個簡單的 `multipath.conf` 如下
```
    blacklist {
        devnode "^sda"
    }
    defaults {
        user_friendly_names yes
        path_grouping_policy multibus
        failback immediate
        no_path_retry fail
    }
```

```
    $ sudo systemctl start multipathd.service
    $ sudo systemctl enable multipathd.service
```

這時候再 `$ lsblk`
```
    NAME        MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
    sda           8:0    0 279.4G  0 disk  
    ├─sda1        8:1    0     1G  0 part  /boot
    └─sda2        8:2    0 278.3G  0 part  
      ├─cl-root 253:0    0    50G  0 lvm   /
      ├─cl-swap 253:1    0  23.6G  0 lvm   [SWAP]
      └─cl-home 253:2    0 204.6G  0 lvm   /home
    sdb           8:32   0  14.5T  0 disk  
    └─mpatha    253:3    0  14.5T  0 mpath 
      └─mpatha1 253:4    0  14.5T  0 part  
    sdc           8:48   0  14.5T  0 disk  
    └─mpatha    253:3    0  14.5T  0 mpath 
      └─mpatha1 253:4    0  14.5T  0 part  
    sr0          11:0    1  1024M  0 rom 
```

可以看到 mpath 出現，然後用 gparted 把 /dev/mapper/mpatha1 格式化(略)

格式化完，`$ sudo mkdir /data` 並在 `/etc/fstab` 最後一行下面加

```
    /dev/mapper/mpatha1     /data   xfs _netdev     0 0
```

用 `$ sudo mount -a` 掛起來看看，用 `$ df -h | grep data` 檢查
```
    /dev/mapper/mpatha1   15T  3.7T   11T  25% /data
```
如果出現上述輸出表示掛成功了 :)

