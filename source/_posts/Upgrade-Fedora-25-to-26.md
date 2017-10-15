---
title: Upgrade Fedora 25 to 26
date: 2017-10-12 17:11:37
tags:
---

<center>Fedora 25 升級到 26<center>
===
<br>

開箱了一台 workstation ，原本要裝 f26 server ，裝到 f25 server 去了，所以就多了這篇 XD

## (設定網路的)坑

裝完 f25 ，馬上就遇到坑，設好 ip (直接改 `/etc/sysconfig/network-scripts/ifcfg-<interface name>`)，連線正常，要 `$sudo dnf update` 卻出現 `錯誤：Failed to synchronize cache for repo 'updates'` 

- 用 `$ sudo dnf update --refresh -vvv` 發現有這行錯誤 `[Could not resolve host: mirrors.fedoraproject.org].` 
- 檢查一下 `/etc/resolv.conf`，原來 NetworkManger 會去改 /etc/resolv.conf，overwrite 成 DHCP 拿到的 domain， nameserver 被洗掉就不能 resolve 鏡像站的 address 
- 解決方法： `sudo vi /etc/NetworkManager/NetworkManager.conf`在 `[main]` 下加一句 `dns:none`

參考：[限制 NetworkManger 不要改 /etc/resolv.conf 的 dns server 設定](https://unix.stackexchange.com/questions/90035/how-to-set-dns-resolver-in-fedora-using-network-manager)

可以更新套件之後，立刻手刀安裝 nmtui ，這有一個很長的名字 network configuration using a text user interface ，我在 CentOS 上已習慣用 nmtui 設定網路(ethernet、bridge)、 hostname 、 ifup/ifdown ，比起手動改設定檔/nmcli/hostnamectl 來的直觀好用，不知道為什麼 f25 當初用光碟裝的時候沒裝

```
    $ sudo dnf install NetworkManager-tui -y
```

裝好 nmtui 之後我就把 `dns:none` 那行拔掉了，直接 `$ sudo nmtui` 把網路設定重新順一下，也 ignore ipv6

---

## f25 to f26

按照[教學](https://fedoraproject.org/wiki/DNF_system_upgrade)，很順利的升到了 f26 ，文件沒什麼問題，就是每一行安裝可以加個 `-y` 參數默認同意以節省時間
```
    $ sudo dnf upgrade --refresh -y
    $ sudo dnf install dnf-plugin-system-upgrade -y
    $ sudo dnf system-upgrade download --refresh --releasever=26 -y
    $ sudo dnf system-upgrade reboot
```

升級完檢查一下版號
```
    $ cat /etc/fedora-release
    Fedora release 26 (Twenty Six)
```

## 其它小習慣

### SELINUX
然後我是習慣把 selinux 關掉，利用 sed 的搜尋(search)並取代(replace)
```
    $ sudo sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
    $ sudo reboot
```

重新開機之後 `$ sestatus` 確定是不是 disabled

### sshd_config
至少把 RootLogin 拿掉
```
    $ sudo sed -i 's/PermitRootLogin\ yes/PermitRootLogin\ no/g' /etc/ssh/sshd_config
    $ sudo systemctl restart sshd.service
```

public key 設好的話 `PasswordAuthentication yes` 也可以設為 `no`

還有很多設定像是 fail2ban 、 vimrc 、 bashrc 之類的…以後再寫，這兩項先做，SELINUX 也沒那麼不值得用，就是我嫌煩

結果主要的部分反而比較簡單…只是又再熟悉了一下網路設定的部分
