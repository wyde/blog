---
title: 網管實用指令-Linux 系統篇
date: 2017-12-12 23:17:47
tags:
- ta217
- System Administration
---
<center>Practical Commands of Linux Administrating</center>
===
<br>

有什麼實用指令就紀錄在這吧

## 傳送單一檔案

source 和 destination 的形式是 `user@host:dir`，傳送是雙向的，source 可以是 local 或 remote ，destination 同理

```
    $ scp <source> <destination> 
```

如果有權限設定，例如說從 server1 的 /etc/blank 傳到 server2 的 /etc

```
    (在 server1 上)
    $ sudo rsync -a -e "ssh" --rsync-path="sudo /rsync" /etc/blank ops@server2:/etc
```

---

## 插入文本最後一行

常用在 export 環境變數到 ~/.bash_profile

```
    $ echo 'source /usr/local/bin/virtualenvwrapper.sh' >> ~/.bash_profile
```

如果要插入有權限的檔案，例如說 mount device 到 /etc/fstab

```
    $ echo '/dev/mapper/... ' | sudo tee --append /etc/fstab > /dev/null
```

---

## grep

找檔案裡有沒有包含特殊的字，` $ grep [選項] [查找模式] [文件1, 文件2...]`

-R: grep recursively
-n: 顯示行號

```
    $ grep -nR "some words" /some/file/path
```

嫌輸出太長又懶得用 awk 裁剪，可以加 -o 只顯示匹配字串，搭配 -e 或 egrep 也很實用

---

## find

我自己最常用的是找符合特定副檔名的檔案，例如 `$ find [dir] *.副檔名`

```
    $ find . *.md
```


