---
title: 如何在 Ubuntu 16.04 和 Fedora 26 上安裝 Docker CE
date: 2017-11-08 14:26:08
tags:
- Ubuntu 16.04
- Fedora 26
- Docker CE
---

CE 就是指 Community Edition，社群貢獻的免費版本，安裝 docker 的步驟其實差不多，紀錄一下 package name 而已
- 清除舊的 docker
- 安裝 docker 的 repo
- 選擇 docker 版本並安裝
- post installation 例如不用 sudo 執行 docker

---

## on Ubuntu 16.04

[官方教學](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)

```
$ sudo apt-get remove docker docker-engine docker.io
$ sudo apt-get update
$ sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
$ sudo apt-get update
```

如果在 production 系統上安裝 docker 最好選擇固定版本，不然就直接用 latest
(optional but recommanded)
```
$ apt-cache madison docker-ce
 docker-ce | 17.09.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.06.2~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.06.1~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.06.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.03.2~ce-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.03.1~ce-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.03.0~ce-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
$ sudo apt-get install docker-ce=17.09.0~ce-0~ubuntu
```

直接安裝 latest
(fast in dev environment)
```
$ sudo apt-get install -y docker-ce
```

post installation
```
$ sudo groupadd docker
$ sudo usermod -aG docker $USER
```

重新登入測試能不能不用 sudo 執行 docker
```
$ docker run hello-world
$ sudo systemctl enable docker
```

若要移除
```
$ sudo apt-get purge docker-ce
$ sudo rm -rf /var/lib/docker
```

---

## on f26

[官方教學](https://docs.docker.com/engine/installation/linux/docker-ce/fedora/) ，只有寫 f24 f25 不過我自己測試適用 f26

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

post installation(這裡跟 ubuntu 一樣)
```
$ sudo groupadd docker
$ sudo usermod -aG docker $USER
```

重新登入測試能不能不用 sudo 執行 docker
```
$ docker run hello-world
$ sudo systemctl enable docker
```

應該會看到歡迎訊息，如果不想用了要解除安裝
```
    $ sudo dnf remove docker-ce
    $ sudo rm -rf /var/lib/docker
```
