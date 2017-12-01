---
title: 如何在 Ubuntu 16.04 上用 Docker 跑 Tensorflow
date: 2017-11-09 12:54:47
tags:
---

<center>How to Install Tensorflow using Docker on Ubuntu 16.04</center>
===
<br>

繼上一篇[在 Ubuntu 16.04 的 Virtualenv 裡安裝 Tensorflow with GPU](https://wyde.github.io/2017/11/07/How-to-Install-Tensorflow-with-GPU-in-Virtualenv-on-Ubuntu-16-04/)，同時也想在該篇用 docker 跑 tf ，原本預期會很順利，結果還是有些小坑...後來還是裝好了

- 硬體：
    - Nvidia 1080 Ti 兩張 with 1000W Power
    - Intel i7 7700K
    - ASUS STRIX Z270F
- 軟體：
    - Ubuntu 16.04 with kernel 4.4
    - Python 3.5 with virtualenv
    - Cuda 8.0 driver updated to 381.22
    - cuDNN v6.0
    - gcc 5.4
    - docker-ce=17.09.0~ce-0~ubuntu


## 裝 GPU driver

如果已經照 virtualenv 那篇作過、裝過 cuda 了，就不用再裝 driver ，如果是要在 docker 裡面跑 tf ， host machine 也不用裝 cuDNN；如果沒裝 cuda 的話，裝 driver 至少要 381 以上的版本才有支援 GTX 1080 ti
```
    $ sudo add-apt-repository ppa:graphics-drivers/ppa
    $ sudo apt update
    $ sudo apt-get install nvidia-381
    $ sudo apt install nvidia-modprobe
```

## 裝 Docker CE(Community Edition)

可以參考我之前的文章[如何在 Ubuntu 16.04 和 Fedora 26 上安裝 Docker Coummunity 版本](https://wyde.github.io/2017/11/08/How-to-Install-Docker-CE-on-Ubuntu-16-04-and-Fedora-26/)

## 裝 nvidia-docker

官方教學在 [nvidia-docker github 的 README.md ](https://github.com/NVIDIA/nvidia-docker)上

```
    $ wget -P /tmp https://github.com/NVIDIA/nvidia-docker/releases/download/v1.0.1/nvidia-docker-1.0.1-1.x86_64.rpm
    $ sudo rpm -i /tmp/nvidia-docker*.rpm && rm /tmp/nvidia-docker*.rpm
    $ sudo systemctl start nvidia-docker
```

這邊裝完有個 [issue](https://github.com/NVIDIA/nvidia-docker/issues/437)，所以要再加兩行
```
    $ sudo service nvidia-docker start
    $ sudo nvidia-docker-plugin
```

由於我是用 cuda8 ，所以在 docker 內測試 nvidia-smi 要用下列指令
```
    $ nvidia-docker run --rm nvidia/cuda:8.0 nvidia-smi
```
應該會跟在 host machine 上直接 `$ nvidia-smi` 得到一樣的結果
![](https://i.imgur.com/dK0dPul.png)

---

## Hello Tensorflow

我們可以用類似下列的指令，從 [tensor flow 的 docker hub](https://hub.docker.com/r/tensorflow/tensorflow/tags/) 拉適合的 binary 來玩，然後透過 jupyter notebook 把玩 tf
``` 
    $ nvidia-docker run -it -p <hostPort:containerPort> <TensorFlowGPUImage> 
```

測試是否安裝完成可以參考[validate your installation](https://www.tensorflow.org/install/install_linux#ValidateYourInstallation)，這邊直接生起一個 jupyter server，壓下下列指令
```
    $ nvidia-docker run -it -p 8888:8888 gcr.io/tensorflow/tensorflow:latest-gpu
```

應該就可以在瀏覽器鍵入 `localhost:8888` 或 `<host machine address or ip>:8888` 看到 jupyter 的起始畫面，然後 terminal 應該會出現類似以下的輸入
```
[C 02:19:50.023 NotebookApp] 
    
    Copy/paste this URL into your browser when you connect for the first time,
    to login with a token:
        http://localhost:8888/?token=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
複製 `token=` 後面那段英數字貼到 jupyter 要求輸入的密碼欄，即可看到 3 個 jupyter notebook 如下列畫面
![](https://i.imgur.com/UxcR38B.png)

可以隨便挑一個跑跑看
