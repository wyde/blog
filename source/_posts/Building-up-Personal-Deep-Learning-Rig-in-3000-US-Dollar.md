---
title: 100k 搭建個人深度學習機
tags:
  - Deep Learning Rig
  - Nvidia GeForce GTX 1080Ti
date: 2017-11-05 14:02:05
---

<center>Building up Personal Deep Learning Rig in 3000 US Dollar</center>
===
<br>

先附上內部實拍一張
![](https://i.imgur.com/ecdzdKU.png)

## 硬體規格

- CPU: Intel Core I7-7700K / 4.2G
- 風扇: INTEL BXTS15A
- M/B: ASUS STRIX Z270F GAMING
- RAM: Kingston 16G DDR4 2400 * 4 = 64G
- SSD: Kingston 240G M.2 2280
- VGA: ASUS TURBO-GTX1080TI-11G * 2
- DVDRW: SATA
- POWER: 1000W, Cooler Master V1000 80Plus 金牌
- 殼: Cooler Master CM6900III
- HD: WD (紅標?) 3TB

---

## 如何安裝 tensorflow

tf 用 python 環境安裝方式主要有 5 種，然後 Python 又分 2.7 和 3.x，排列組合一下吧… 
- virtualenv
- "native" pip
- Docker                                                                           
- Anaconda
- install from source 

### 成功安裝 tensorflow 的方式
Tensorflow 有許多裝法，其他裝法如果後續有嘗試我會在下面補充
- (2017.11.08 更新) [Ubuntu 16.04 + Virtualenv](https://wyde.github.io/2017/11/07/How-to-Install-Tensorflow-with-GPU-in-Virtualenv-on-Ubuntu-16-04/)
- (2017.11.09 更新) [Ubuntu 16.04 + Docker + Jupyter](https://wyde.github.io/2017/11/09/How-to-Install-Tensorflow-using-Docker-on-Ubuntu-16-04/)
- (2017.11.09 更新) [Fedora 26 + Docker](https://wyde.github.io/2017/11/09/How-to-Install-Tensorflow-using-Docker-on-Fedora-26/)

### 安裝失敗的經驗參考

- f26 + gcc7 + cuda9 + cudnn7 (failed) 
    在寫這篇文章的時候，已經出了 cuda 9 和 cudnn v7.0，雖然[ f27 也號稱要支援這兩者](https://negativo17.org/cuda-9-0-cudnn-7-0-and-wayland-support-in-fedora-27/)，我還
是先用 f26 試試看，殊不知 f26 預設已經到了 gcc 7，cuda 9 不支援，其實對照下面的圖，我根本不該有[在 f26 上安裝 virtualenv 版的 tf 會很順利的錯覺]…當初還花時間[從 f25 升
到 f26](https://wyde.github.io/2017/10/12/Upgrade-Fedora-25-to-26/)，我相信一定有辦法可以裝，只是都要出 f27 了，我不想折騰
- f25 + gcc6 + cuda9 (failed)
    ok，重灌成 f25 應該可以了吧，按照[cuda installtion guide](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html)這次 cuda 9 裝好了，殊不知測試安裝成
功時報錯 `ImportError: libcublas.so.8.0: cannot open shared object file: No such file or directory`，我知道這看起來很像 `export LD_LIBRARY_PATH`之類，像是[這篇文章](https://stackoverflow.com/questions/41746576/tensorflow-with-cuda-importerror)可以解決的事，但怎麼設怎麼失敗
- f25 negativo17 (failed)
    再次重灌，然後有查到這篇文章，[Nvidia (8.0) installation for TensorFlow on Fedora 25](http://blog.mdda.net/oss/2016/11/25/nvidia-on-fedora-25)，想說雖然是 python2.7 還是試試看，裝起來進 python shell 會報 Segmentation Fault...現在想起來可能是我的 python 環境指到 python3 了，不過我想盡可能不用第三方的 repo，這次也是果斷棄坑
- f25 + gcc6 + cuda8 + cudnn6 (failed)
    再次重灌，回頭再重看一次 tensorflow 的安裝，發現它requirement 寫的是 cuda 8.0、cudnn v6.0，想說用這個版本裝裝看，在 cuda archive 裡找到 cuda8，裝完之後，好不容易找
到 cudnn v6.0 的 archive(真的很隱晦的位置...)，發現官方只有提供 *.deb.........要不要這麼坑，如果有 rpm 版的也許我會考慮再試一次...
![](https://i.imgur.com/fh3SGJG.png)
