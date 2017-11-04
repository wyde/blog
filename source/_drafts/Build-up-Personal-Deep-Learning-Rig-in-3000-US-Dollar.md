---
title: Build up Personal Deep Learning Rig in 3000 US Dollar
tags:
- Deep Learning Rig
- Nvidida 1080Ti
---
<center>100k 搭建個人深度學習機</center>
===
<br>

先附上沒插卡的 ASUS STRIX Z270F GAMING 主機板玉照一張
![](https://i.imgur.com/7mkBlHE.png)

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

## Overview

我的目標是 硬體組好、 Hello Tensorflow 跑起來，host OS 會使用 fedora，[官方教學只有 Ubuntu](https://www.tensorflow.org/install/install_linux) 的，大概是覺得 Ubnuntu 是最多人用的發行版，但還是可以參考一下，至於為什麼用 fedora ，純粹就是個人偏好 RPM

tf 用 python 環境安裝方式主要有 5 種，然後 Python 又分 2.7 和 3.x，排列組合一下吧…
- virtualenv
- "native" pip
- Docker
- Anaconda
- install from source

因為要來寫 [ud730](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/examples/udacity) 的作業當作練習(天啊我超 lag)，作業是以 docker 的方式給的，所以一開始的安裝方式就會以 docker 的方式處理。稍微看了一下[官方教學](https://www.tensorflow.org/install/install_linux#InstallingDocker)，所以順序大概會是

- 裝 GPU 驅動
- 裝 Docker CE(Community Edititon) for Fedora
- 裝 nvidia-docker](https://github.com/NVIDIA/nvidia-docker
- 跑 Hello World

後來我也把 virtualenv 的方式裝了一遍，在本文最下面

---

## 裝 GPU 驅動

放進來篇幅太長，請參考我上一篇筆記[在 Fedora 上安裝 Nvidia 1080Ti 的驅動](https://wyde.github.io/2017/11/03/How-to-Install-Nvidia-1080Ti-GPU-Driver-on-Fedora/)

---

## 裝 Docker CE

可以參考[官方教學](https://docs.docker.com/engine/installation/linux/docker-ce/fedora/)，其實只有幾行，先處理一些前置作業，安裝 repo
```
    $ sudo dnf remove docker docker-common docker-selinux docker-engine-selinux docker-engine
    $ sudo dnf -y install dnf-plugins-core
    $ sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
```

然後安裝 docker
```
    $ sudo dnf install -y docker-ce
    $ dnf list docker-ce  --showduplicates | sort -r
    docker-ce.x86_64               17.09.0.ce-1.fc26               docker-ce-stable
```

官方建議在 production 系統上應該要安裝同一版本的 docker-ce，所以安裝時要給版本參數
```
    $ sudo dnf -y install docker-ce-17.09.0.ce-1.fc26
    $ sudo systemctl start docker; sudo systemctl enable docker
    $ sudo docker run hello-world
```

應該會看到歡迎訊息，如果不想用了要解除安裝
```
    $ sudo dnf remove docker-ce
    $ sudo rm -rf /var/lib/docke
```

---

## 裝 nvidia-docker

官方教學在 [nvidia-docker github 的 README.md ](https://github.com/NVIDIA/nvidia-docker)上

```
    # Install nvidia-docker and nvidia-docker-plugin
    $ wget -P /tmp https://github.com/NVIDIA/nvidia-docker/releases/download/v1.0.1/nvidia-docker-1.0.1-1.x86_64.rpm
    $ sudo rpm -i /tmp/nvidia-docker*.rpm && rm /tmp/nvidia-docker*.rpm
    $ sudo systemctl start nvidia-docker
    
    # Test nvidia-smi
    $ nvidia-docker run --rm nvidia/cuda nvidia-smi
```

最後一行 # Test nvidia-smi 會跟 `$ nvidia-smi` 有一樣的輸出，就是裝 GPU driver 時曾經出現的這張
![](https://i.imgur.com/RcKpQl8.png)

---

## Hello Tensorflow

我們可以用類似下列的指令，從 [tensor flow 的 repo](https://hub.docker.com/r/tensorflow/tensorflow/tags/) 拉適合的 binary 來玩 
``` 
    $ nvidia-docker run -it -p <hostPort:containerPort> <TensorFlowGPUImage> 
```

[validate your installation](https://www.tensorflow.org/install/install_linux#ValidateYourInstallation)
```
    $ docker run -it gcr.io/tensorflow/tensorflow bash
    Unable to find image 'gcr.io/tensorflow/tensorflow:latest' locally
    latest: Pulling from tensorflow/tensorflow
    f6fa9a861b90: Pull complete 
    da7318603015: Pull complete 
    6a8bd10c9278: Pull complete 
    d5a40291440f: Pull complete 
    bbdd8a83c0f1: Pull complete 
    abeb898c5fe7: Pull complete 
    d66940709c30: Pull complete 
    44835d776a0b: Pull complete 
    6ec5364eed97: Pull complete 
    3ff1cc6e638b: Pull complete 
    56aa69af61b9: Pull complete 
    982e3b05055e: Pull complete 
    Digest: sha256:581e048541034a6992d85cfd37fd36e8085d34142bcb7f57a2a680dc96e38f7e
    Status: Downloaded newer image for gcr.io/tensorflow/tensorflow:latest
    root@47a80733b08c:/notebooks#
```

進入 docker 之後，進入 python interactive shell
```
    >>> import tensorflow as tf
    >>> hello = tf.constant('Hello, TensorFlow!')
    >>> sess = tf.Session()
    >>> print(sess.run(hello))
    Hello, TensorFlow!
```

docker 的安裝完成，可以從以下兩種方式來入門
- [Udacity deep-learning--ud730](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/examples/udacity)  
- [Getting Started with Tensorflow](https://www.tensorflow.org/get_started/get_started)

---

## 用 virtualenv 裝一次

用 virtualenv 安裝是[官方推薦的方式](https://www.tensorflow.org/install/install_linux#InstallingVirtualenv)，想想有時候寫個小 script 也要用 docker 似乎也是麻煩，就順手來裝一下， virtualenv 的設定可以參考我之前寫的 [CentOS 7 上設定 virtualenv 的方式](https://wyde.github.io/2017/10/30/Setting-up-Python-3-Virtual-Environment-on-CentOS-7/)，跟 Fedora 基本上是一樣的

```
    $ sudo dnf install -y python3-pip python3-devel python-virtualenv python-virtualenvwrapper
    $ echo "export WORKON_HOME=~/.virtualenvs" >> ~/.bashrc
    $ source /usr/bin/virtualenvwrapper.sh
    $ mkvirtualenv --python=`which python3` tensorflow
    $ pip3 install --upgrade tensorflow-gpu
```

然後還要裝 cuda，先到 [cuda 的網站](https://developer.nvidia.com/cuda-downloads)選擇符合自己作業系統的安裝包(如下圖)，我是 f26 ，不過我猜 f25 的包應該也可以裝
![](https://i.imgur.com/xQx6KJI.png)

```
    $ wget https://developer.nvidia.com/compute/cuda/9.0/Prod/local_installers/cuda-repo-fedora25-9-0-local-9.0.176-1.x86_64-rpm -P /tmp
    $ sudo rpm -i /tmp/cuda-repo-fedora25-9-0-local-9.0.176-1.x86_64-rpm
    $ sudo dnf clean all
    $ sudo dnf install cuda
```

OK，這樣就裝完了， validation 跟上面的 code 一樣就不再重複貼了，差別在於上面 docker 拉的版本是 python2.7 CPU 版(GPU 版應該要用 nvidia-docker)，而這邊 virtualenv 用的是 python3.6 GPU 版，應該說寫作業的話就看課堂給的 docker 所附的 python 版本是什麼都可以，生產環境應該會用 python3，某種程度上這也是 docker 的好處 :)
