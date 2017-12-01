---
title: 如何在 f26 上用 Docker 跑 Tensorflow 
tags:
  - f26
  - Fedora
  - Tensorflow
  - Docker
date: 2017-11-09 12:54:36
---

<center>How to Install Tensorflow using Docker on Fedora 26</center>
===
<br>

## 裝 GPU 驅動

放進來篇幅太長，請參考我之前的筆記[在 Fedora 上安裝 Nvidia 1080Ti 的驅動](https://wyde.github.io/2017/11/03/How-to-Install-Nvidia-1080Ti-GPU-Driver-on-Fedora/)

---

## 裝 Docker CE

參考
- [官方教學](https://docs.docker.com/engine/installation/linux/docker-ce/fedora/)
- 我的筆記[如何在 Ubuntu 16.04 和 Fedora 26 上安裝 Docker Coummunity 版本](https://wyde.github.io/2017/11/08/How-to-Install-Docker-CE-on-Ubuntu-16-04-and-Fedora-26/#more)

其實只有幾行，先處理一些前置作業，安裝 repo
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

