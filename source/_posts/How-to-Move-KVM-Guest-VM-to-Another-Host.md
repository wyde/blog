---
title: 搬 KVM 虛擬機
date: 2017-12-13 17:21:19
tags:
---
<center>How to Move KVM Guest VM to Another Host</center>
===
<br>

Guest VM 在 Host 之間搬來搬去也是蠻常見的事，至少在我們重構 legacy 架構的時候很常用，如果基礎建設同質性越高越容易

## 實驗環境

- 兩台 Linux host: host1 、 host2
- 掛 image 的 partition 要有足夠空間
    - qcow2 size: 50G
    - /image/qcow
- 同樣的網路架構
    - bridge: br3 (第 3 個 interface 作 bridge)
    - 兩台 host 各拿一個與 guest vm 相同 vlan 的 ip
    - host1: 192.168.31.211
    - host2: 192.168.31.213
    - guest: 192.168.31.217
- domain name of guest VM
    - web-test
- 管理者帳號
    - sysadmin

## on host1

這不是 online 的操作，vm 需要下線，不過時間不會太久，depend on qcow2 image 傳輸的時間
```
    $ sudo virsh shutdown web-test
    $ sudo virsh dumpxml > /tmp/web-test.xml
    $ scp /tmp/web-test.xml host2:/tmp/web-test.xml
    $ sudo rsync -a -e "ssh" --rsync-path="sudo rsync" /image/qcow2/web-test.qcow2 sysadmin@host2:/image/qcow2
    
    (optional)
    $ sudo virsh undefine web-test
    $ sudo rm -f /image/qcow2/web-test.qcow2
```

## on host2

如果網路環境不一樣，要先修改 web-test.xml ，確定可以符合 host2
```
    $ sudo virsh define /tmp/web-test.xml
    $ sudo virsh start web-test
```

## Reference

- [KVM - Move Guest to Another Host](http://ostolc.org/kvm-move-guest-to-another-host.html)
