---
title: How to Setup Virtualenv on Ubuntu 16.04
date: 2017-11-07 14:34:05
tags:
- python3
- virtualenv
- virtualenvwrapper
- ubuntu16.04
---
<center>在 Ubuntu 16.04 上建立 Virtualenv</center>
===
<br>

前作[在 CentOS 7 上創建 Python 3 虛擬環境](https://wyde.github.io/2017/10/30/Setting-up-Python-3-Virtual-Environment-on-CentOS-7/#more)，其實在 Ubuntu 上也是大同小異，這篇速記一下順序和 package，同樣也是用原生的 python 2.7 pip、virtualenv、virtualenvwrapper 來建立 python3 的開發環境

```
    $ sudo apt update
    $ sudo apt install -y python-pip python-virtualenv python3-dev
    $ sudo pip install virtualenvwrapper
    $ echo 'export WORKON_HOME="$HOME/.virtualenvs"' >> ~/.bashrc
    $ source ~/.bashrc
    $ source /usr/local/bin/virtualenvwrapper.sh
```

然是是使用 virtualwrapper 來建立環境
```
    $ mkvirtualenv --system-site-packages --python=`which python3` <env name> # 建立
    $ deactivate # 退出
    $ workon <env name> # 進入
    $ rmvirtualenv <env name> # 刪除
    $ lsvirtualenv # 列表
```
