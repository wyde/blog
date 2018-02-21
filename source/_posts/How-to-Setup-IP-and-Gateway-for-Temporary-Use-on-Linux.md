---
title: 在 linux 上暫時設置 IP 和 Gateway 的方法
date: 2017-12-14 18:59:19
tags:
- ta217
- sysadmin
---
<center>How to Setup IP and Gateway for Temporary Use on Linux</center>
===
<br>

這件事在初始化伺服器和測試網路時常常用呢

## 筆記

```
    $ ip a                                  # ip 列表
    $ ip -4 a                               # ipv4 列表，未取得 ipv4 位址則不顯示
    $ ip -4 a sh eno1                       # 列出 device eno1 的 ipv4 位址
    $ ip a add 10.2.17.15/24 dev eno1       # device eno1 加上 ip 且 mask 為 255.255.255.0
    $ ip a del 10.2.17.15/24 dev eno1       # 移除
    $ ip link set dev eno1 up
    $ ip link set dev eno1 down

    $ ip r                                  # show routing rule 
    $ route del default                     # delete default route
    $ route add default gw 10.2.17.254      # add default route
---

## Reference
- [Linux 暫時設定 ip 與 gateway 的方法](http://kenlosolid.blogspot.tw/2013/01/linux-ip-gateway.html)
