---
title: How to Mount a Second Drive Managed by LVM
date: 2017-10-13 19:47:02
tags:
- LVM
- CentOS 7
- ta217
- system administration
- 機房管理
---

<center>用 LVM 管理 CentOS 7 掛載的資料碟<center>
===
<br>

好的，現在的狀況是我們有一台 Linux Server 上面有兩顆作過 RAID 的虛擬硬碟 (VD，Virtual Disk)，我們已經把 OS 灌在容量較小的 VD 當作系統碟 ，要掛載另一顆 VD 當作資料碟，往後的用途就是拿來掛虛擬機的映象檔(image)，所以會掛在根目錄下的 /image

我們會使用 LVM 來管理硬碟空間，LVM 是什麼？使用 LVM 的好處是什麼？請參考[鳥哥](http://linux.vbird.org/linux_basic/0420quota.php#lvm)或是[這篇](https://blog.nuface.tw/?p=1267)也寫得不錯，使用 LVM 的原因主要是著眼未來「動態調整 file system 大小的能力」，這邊就不講原理了，以實作筆記為主

總之在掛這顆資料碟之前，對下列提示要有些了解

- 超過 2TB 的硬碟不能使用 MBR 分割表，要用 GPT
- 知道 LVM 裡 PV、PE、VG、LV 的階層與意義(底下會稍微描述一下)
- 知道 mount point 是什麼概念
- 對 file system 裡的 ext 家族與 xfs 有一點概念

# 實驗環境

- Dell PowerEdge R710 * 1
- 系統碟：450GB 3.5吋 SAS HDD * 2 (RAID 1)
- 資料碟：2TB 3.5吋 SAS HDD * 4 (RAID 10)
- 作業系統： CentOS 7 (其它發行版掛載的步驟應該大同小異)

## 流程

所以我們要做的事，就是把另一顆(/dev/sdb)大概 3.7TB 做好硬體 raid 的硬碟

1. 建立 GPT 分割表，不切 partition
2. 使用 LVM 管理 logic volume
    - 標示 PV(Physical Volume)
    - 集成 VG(Volume Group)
    - 分割 LV(Logical Volume)
3. 格式化成 xfs 
4. 掛在 /image 目錄下

如果確定未來不會動態調整 partition ，可以不用 LVM ，直接跳過 2 就可以了。LVM 的概念，我自己的理解是每顆 VD 可以標示成 PV 為 LVM 所用，一個或多個 PV 可以結合成 VG，在此基礎上可以分割粒度更小的 LV

因為我們目前規劃是整顆 VD 都要拿來放 image ，所以空間的粒度是一樣大的， VD=PV=VG=LV ，假設未來需要 resize ，就把屬於 image 的 LV 縮小，來容納新的 LV

LVM 與 非 LVM 系統概念上的對應
- PV <-> RAID
- VG <-> disk
- LV <-> partition

---

## 使用 parted 建立 GPT 分割表

```
    $ sudo parted /dev/sdb
    GNU Parted 3.1

    (parted) mklabel gpt
    (parted) mkpart primary xfs 0GB 100%
```

這樣就建完了，`print`一下看結果
```
    (parted) print
    型號：DELL PERC H700 (scsi)
    磁碟 /dev/sdb：4000GB
    磁區大小 (邏輯/物理)：512B/512B
    分割區：gpt 
    Disk Flags: 
    
    編號  起始點  結束點  大小    檔案系統  名稱     旗標
     1    1049kB  4000GB  4000GB            primary
    
    (parted) q
```

查看 block device
```
    $ lsblk
    sdb           8:16   0   3.7T  0 disk 
    └─sdb1        8:17   0   3.7T  0 part
```

---

## 使用 LVM

### PV

```
    $ sudo pvcreate /dev/sdb1
      Physical volume "/dev/sdb1" successfully created.
    
    $ sudo pvs
      PV         VG Fmt  Attr PSize   PFree 
      /dev/sda2  cl lvm2 a--  417.62g  4.00m
      /dev/sdb1     lvm2 ---   <3.64t <3.64t
```

### VG

這裡我把 VG 的 name 叫做 `vg-image`
```
    $ sudo vgcreate vg-image /dev/sdb1
      Volume group "vg-image" successfully created
    
    $ sudo vgs
      VG       #PV #LV #SN Attr   VSize   VFree 
      cl         1   3   0 wz--n- 417.62g  4.00m
      vg-image   1   0   0 wz--n-  <3.64t <3.64t
```

### LV

-n: name 這裡命名為 `lv-image`
-l: 該 LV 所佔 VG 比例
-L: fixed size

```
    $ sudo lvcreate -n lv-image -l 100%FREE vg-image
      Logical volume "lv-image" created.
    
    $ sudo lvs
      LV       VG        Attr     LSize
      home     cl       -wi-ao---- <336.12g
      root     cl       -wi-ao----   50.00g
      swap     cl       -wi-ao----   31.50g
      lv-image vg-image -wi-a-----   <3.64t
```

到這就做完 LVM 的設定，可以 `$ sudo lvdisplay <vg name>/<lv name>` 檢視 detail
- 檢視 list：pvs、vgs、lvs
- 檢視 detail：pvdisplay、vgdisplay、lvdisplay

## 格式化

這時候 LV 的路徑在 `/dev/<vg name>/<lv name>`，所以就直接
```
    $ sudo mkfs.xfs /dev/vg-image/lv-image
```
過個幾秒應該就格式化好了

---

## 掛載(mount)

我們要把 `lv-image` 掛到 `/image` 目錄，然後寫進 `/etc/fstab` 讓系統開機自動 mount，這邊的 UUID 要看個別的系統而定

```
    $ sudo mkdir /image
    $ sudo blkid /dev/vg-image/lv-image
    /dev/vg-image/lv-image: UUID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" TYPE="xfs"

    $ echo `sudo blkid /dev/vg-image/lv-image | awk '{print $2}'` ' /image xfs    defaults        0 0' | sudo tee --append /etc/fstab > /dev/null
    $ sudo mount -a
```

如果要 `$ sudo echo "some words" > /<priviledged file path>`，是行不通的，該技巧可見[討論](https://stackoverflow.com/questions/84882/sudo-echo-something-etc-privilegedfile-doesnt-work-is-there-an-alterna)

`df` 一下
```
    $ df -h | grep /image
    /dev/mapper/vg-image-lv-image  3.7T   33M  3.7T    1% /image
```

到這邊，就成功把資料碟以 LVM 管理的方式掛載起來了

## Reference

- [How to Manage and Create LVM Using vgcreate, lvcreate and lvextend Commands ](https://www.tecmint.com/manage-and-create-lvm-parition-using-vgcreate-lvcreate-and-lvextend/)
- [Linux Creating a Partition Size Larger Than 2TB](https://www.cyberciti.biz/tips/fdisk-unable-to-create-partition-greater-2tb.html)
