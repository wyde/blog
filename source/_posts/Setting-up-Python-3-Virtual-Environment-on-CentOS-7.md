---
title: Setting up Python 3 Virtual Environment on CentOS 7
date: 2017-10-30 19:24:19
tags:
- virtualenv
- virtualenvwrapper
- pip
- python 2.7
- python 3.4
---
<center>在 CentOS 7 上創建 Python 3 虛擬環境<center>
===
<br>

CentOS 7 上沒有自帶 python 3，`$ which python` 可以看到預設是 python 2.7，`$ ls -l /usr/bin/ | grep python`，也沒看到 python3  ，我不打算變更系統設定，打算另外安裝 python 3.4 在虛擬環境裡使用

記得以前還用過 easy_install，現在 python 的 package management 應該都果斷用 pip 了吧，如果把預設的 python 換成 python3，python 3.4 之後就能用內建 pyenv 直接開虛擬環境，不過就像我最前面說的，我不想更動預設的 python 2.7 ，所以還是會裝 virtualenv ，至於 virtualenvwrapper ，是有點 overkill ，我時裝時不裝，為了方便其他人一起共管 hosting server ，還是裝了， 畢竟 `$ workon ` + <tab> 進入虛擬環境的方式，還是可以在交接時少說兩句廢話

整理一下， python 2 與 python 3 的包管理，其實講到包管理，應該把 miniconda 拉進來比較，懶得製表就先條列來比較一下

- python 2.7
    - pip
    - virtualenv, virtualenvwrapper
- python 3 (3.4 for here)
    - pip3
    - pyenv

> 結論就是，先裝 python 3 ，用 python 2.7 的 virtualenv 開 python 3.4 的虛擬環境

## installation of  python3.4

如果要安裝 python 3 ，常見的選擇有

1. compile from source [參考連結](https://linuxconfig.org/compile-and-install-python-3-on-centos-7-linux-from-source)
2. install binaries from [EPEL](https://fedoraproject.org/wiki/EPEL)
3. install binaries from [IUS Community](https://ius.io/)

我選擇第 2 種方法，原因是懶得編第 1 種方法了，還有等一下要安裝的 virtualenv 也可以從 epel 的 repo 拉
```
    $ sudo yum update -y
    $ sudo yum install -y epel-release python34
```

檢查一下 
```
    $ which python3
    /usr/bin/python3
```

## installation of  pip, virtualenv and virtualenvwrapper

```
    $ sudo yum install -y python-pip python-virtualenv python-virtualenvwrapper
```

(optional) 如果不想用 wrapper ，可以直接 `$ virtualenv -p python3 ~/.env/` ，建立不受 wrapper 管理的 virtualenv

設定 virtaulenvwrapper 的環境變數
```
    $ export WORKON_HOME=~/.virtualenvs
    $ source /usr/bin/virtualenvwrapper.sh
```
這裡用全局更新了 pip 和安裝 virtualenv ，所以需要 sudo 權限，如果沒有高權限，那就只能 `--user` 在自己的家目錄下

## 指令

創建 python 3 虛擬環境
```
    $ mkvirtualenv --python=`which python3` <env name>
```

退出虛擬環境
```
    $ deactivate
```

進入虛擬環境
```
    $ workon <env name>
```

其他指令
```
    $ lsvirtualenv # 列出所有虛擬環境
    $ rmvirtualenv <env name> # 刪除某個虛擬環境
```

