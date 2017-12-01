---
title: 如何在 Ubuntu 16.04 的 Virtualenv 裡安裝 Tensorflow with GPU
tags:
  - ubuntu16.04
  - python3.5
  - virtualenv
  - tensorflow
date: 2017-11-07 17:21:01
---

TL;DR 安裝 Tensorflow 要麼直接用 docker ，要麼用 ubuntu ，用其他發行版搞起 dependency 我覺得很麻煩...
<br>
#重灌的乾淨ubuntu 
#不用先手動裝驅動 
#簡單快速不易失敗 
#盡量使用官方套件與教學
<br>
前情提要：之前已經寫了一個[100k 搭建深度學習個人環境](https://wyde.github.io/2017/11/05/Building-up-Personal-Deep-Learning-Rig-in-3000-US-Dollar/)，用 f26 + docker 裝得順順的，想說順手來裝個 python virtualenv 的版本，沒想到踩到了大坑...最後還是裝好了，用的是以下版本

- 硬體：
    - Nvidia 1080 Ti 兩張 with 1000W Power
    - Intel i7 7700K
    - ASUS STRIX Z270F
- 軟體：
    - Ubuntu 16.04 with kernel 4.4
    - Python 3.5 with virtualenv
    - Cuda 8.0
    - cuDNN v6.0
    - gcc 5.4
    - tensorflow-gpu 1.4.0
    - driver ~~375.26~~ 381.22

---

根據 [Tensorflow 在 Ubuntu 上的安裝教學](https://www.tensorflow.org/install/install_linux) ，大概分成幾個部分，
- 首先要有硬體(廢話)，然後要有一張 Compute Capability 3.0 以上的 GPU card，[查詢頁面](https://developer.nvidia.com/cuda-gpus)，可以看到 GeForce GTX 1080 Ti 是 6.1 分
- 安裝 CUDA Toolkit 8.0 ，[官方安裝教學](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/#axzz4VZnqTJ2A)，[下載位址](https://developer.nvidia.com/cuda-80-ga2-download-archive)，然後記得設 LD_LIBRARY_PATH 環境變數
- 安裝 cuDNN v6.0 ，[官方安裝教學](http://docs.nvidia.com/deeplearning/sdk/cudnn-install/index.html)，[下載位址(需免費登入)](https://developer.nvidia.com/rdp/cudnn-download)，記得設 CUDA_HOME 環境變數

以下分步驟

---

## 安裝 apt 套件

```
    $ sudo apt install -y gcc make linux-headers-$(uname -r) libcupti-dev
```

## 下載並安裝 cuda8

至[官網](https://developer.nvidia.com/cuda-80-ga2-download-archive)選擇 target platform 如下圖，然後 installer type 選 runfile(local)
![](https://i.imgur.com/SOlBeAA.png)

```
    $ wget https://developer.nvidia.com/compute/cuda/8.0/Prod2/local_installers/cuda_8.0.61_375.26_linux-run -P ~ # 1.4GB
    $ sudo sh ~/cuda_8.0.61_375.26_linux.run
```
全部都按 yes 或 default 就可以了，第一次可能會安裝失敗，因為 installer 要把 nouveau (開源顯卡驅動) 關掉(設定會寫在 ` /etc/modprobe.d/nvidia-installer-disable-nouveau.conf`，如果有成功把 nouveau 列入 blacklist ，系統需要 reboot

接著裝 patch
```
    $ wget https://developer.nvidia.com/compute/cuda/8.0/Prod2/patches/2/cuda_8.0.61.2_linux-run -P ~ # 95.3 MB
    $ sudo sh ~/cuda_8.0.61.2_linux-run
```
如果前一步有裝好，這步應該很快

---

(2017/11/9 更新)

## 更新 driver

根據[這篇文章](https://blog.nelsonliu.me/2017/04/29/installing-and-updating-gtx-1080-ti-cuda-drivers-on-ubuntu/)， cuda8 的 driver 是 375.26 ，不支援 GTX 1080 ti，所以我們要升到 381 ，可以透過 `$ nvidia-smi` 指令查看 driver 的訊息
```
    $ sudo add-apt-repository ppa:graphics-drivers/ppa
    $ sudo apt update
    $ sudo apt-get install -y nvidia-381
    $ sudo apt-get install -y nvidia-modprobe
```

大概就是一些加 repo 、更新 repo、安裝套件的過程，然後壓下 `$ nvidia-smi` 可以看到更新後的顯卡訊息
![](https://i.imgur.com/dK0dPul.png)

---

## 下載並安裝 cuDNN v6.0

> Prerequisites:
> CUDA 7.5 or higher version and a GPU of compute capability 3.0 or higher are required.

[cuDNN 的 Archive ](https://developer.nvidia.com/rdp/cudnn-archive)只有從 v6.5 開始列，我們要的 v6.0 要先申請 Nvidia Developer 的帳號，然後根據[論壇的帖子](https://devtalk.nvidia.com/default/topic/1023497/no-link-to-download-cudnn-v6-or-v6-1/)找到 [v6.0 版本的下載位置](https://developer.nvidia.com/rdp/cudnn-download)，當初也是找了一陣子，截圖紀念一下

![](https://i.imgur.com/P51gjfs.png)

我下載的路徑在家目錄，這邊請隨意指定想要放的位置，下載好依序安裝即可
```
    $ sudo dpkg -i ~/libcudnn6_6.0.21-1+cuda8.0_amd64.deb # runtime library
    $ sudo dpkg -i ~/libcudnn6-dev_6.0.21-1+cuda8.0_amd64.deb # developer library
    $ sudo dpkig -i ~/libcudnn6-doc_6.0.21-1+cuda8.0_amd64.deb # document library
```

---

## 設定環境變數

```
    $ echo /usr/local/cuda-8.0/lib64 | sudo tee -a /etc/ld.so.conf.d/cuda-8.0.conf > /dev/null
    $ sudo ldconfig
    $ echo 'export CUDA_HOME=/usr/local/cuda-8.0' >> ~/.bash_profile
    $ echo 'export LD_LIBRARY_PATH=${CUDA_HOME}/lib64:$LD_LIBRARY_PATH' >> ~/.bash_profile
    $ echo 'export PATH=${CUDA_HOME}/bin:${PATH}' >> ~/.bash_profile
    $ source ~/.bash_profile
```

---

## 設定 virtualenv 並安裝 tensorflow

我使用 virtualenv + virtualenvwrapper 的組合建立 python3 的開發環境，由於 Ubuntu 16.04 自帶 python 3.5 ，我也跟著用 3.5，每個人建立 virtualenv 的習慣不一樣，也許有的人習慣用 native 的環境，如果要參考我的環境，可以看這篇前作，[在 Ubuntu 16.04 上建立 Virtualenv](https://wyde.github.io/2017/11/07/How-to-Setup-Virtualenv-on-Ubuntu-16-04/)，總之 `$ python --version` 這裡要給一個 3.5，然後我們可以在 [tensorflow 的安裝頁面](https://www.tensorflow.org/install/install_linux#the_url_of_the_tensorflow_python_package)找到對應的 binary url

```
    $ pip3 install --upgrade pip
    $ pip3 install --upgrade https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow_gpu-1.4.0-cp35-cp35m-linux_x86_64.whl 
```

如果是用新的 Ubuntu 16.04 安裝，很有可能也會遇到語系設定的問題，pip install 會報錯，可以參考[在 Ubuntu 16.04 中設定 locale 環境變數](https://wyde.github.io/2017/11/07/How-to-Setup-Locale-in-Ubuntu-16-04/)

## Hello Tensorflow

進入 python interactive shell，按照[官網提供的 validate your installation](https://www.tensorflow.org/install/install_linux#ValidateYourInstallation) 
```
    $ python
    Python 3.5.2 (default, Sep 14 2017, 22:51:06) 
    [GCC 5.4.0 20160609] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import tensorflow as tf
    >>> hello = tf.constant('Hello, TensorFlow!')
    >>> sess = tf.Session()
    2017-11-09 11:18:52.434919: I tensorflow/core/platform/cpu_feature_guard.cc:137] Your CPU supports instructions that this TensorFlow binary was not compiled to use: SSE4.1 SSE4.2 AVX AVX2 FMA
    2017-11-09 11:18:52.587915: I tensorflow/stream_executor/cuda/cuda_gpu_executor.cc:892] successful NUMA node read from SysFS had negative value (-1), but there must be at least one NUMA node, so returning NUMA node zero
    2017-11-09 11:18:52.588325: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1030] Found device 0 with properties: 
    name: GeForce GTX 1080 Ti major: 6 minor: 1 memoryClockRate(GHz): 1.582
    pciBusID: 0000:01:00.0
    totalMemory: 10.91GiB freeMemory: 10.75GiB
    2017-11-09 11:18:52.690694: I tensorflow/stream_executor/cuda/cuda_gpu_executor.cc:892] successful NUMA node read from SysFS had negative value (-1), but there must be at least one NUMA node, so returning NUMA node zero
    2017-11-09 11:18:52.691144: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1030] Found device 1 with properties: 
    name: GeForce GTX 1080 Ti major: 6 minor: 1 memoryClockRate(GHz): 1.582
    pciBusID: 0000:02:00.0
    totalMemory: 10.91GiB freeMemory: 10.75GiB
    2017-11-09 11:18:52.691703: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1045] Device peer to peer matrix
    2017-11-09 11:18:52.692064: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1051] DMA: 0 1 
    2017-11-09 11:18:52.692072: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1061] 0:   Y Y 
    2017-11-09 11:18:52.692076: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1061] 1:   Y Y 
    2017-11-09 11:18:52.692082: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1120] Creating TensorFlow device (/device:GPU:0) -> (device: 0, name: GeForce GTX 1080 Ti, pci bus id: 0000:01:00.0, compute capability: 6.1)
    2017-11-09 11:18:52.692087: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1120] Creating TensorFlow device (/device:GPU:1) -> (device: 1, name: GeForce GTX 1080 Ti, pci bus id: 0000:02:00.0, compute capability: 6.1)

    >>> print(sess.run(hello))
    b'Hello, TensorFlow!'
```

安裝完畢，比 fedora 順利多了，不愧是官方支援…接下來會裝一遍 docker 版的，預期不會遇到太大困難…吧 :)

[Getting started with Tensorflow](https://www.tensorflow.org/get_started/get_started)
