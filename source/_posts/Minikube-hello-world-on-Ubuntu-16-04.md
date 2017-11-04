---
title: Minikube hello world on Ubuntu 16.04
date: 2017-11-01 12:31:50
tags:
- Kubernetes
- k8s
- Minikube
- Ubuntu
- Devops
---
<center>在 Ubuntu 16.04 上玩 Minikube</center>
===
<br>

就像玩 OpenStack 在本機有 DevStack ，如果要玩 Kubernetes 在本機有 Minikube (每次拼 Kubernetes 都怕拼錯，還是叫 k8s 吧)，這篇就來總結一下本機簡單跑 Minikube 起來的筆記

## Concept

先來看一下可愛的架構圖
![](http://omerio.com/wp-content/uploads/2015/12/kubernetes_cluster.png)

[圖片來源](http://dockone.io/article/932)，[這篇文章](http://time-track.cn/Kubernetes-resources-summaries.html)也有蠻好的說明，我白話地寫幾條重點

- 我有一些輕量的 Linux 容器組成微服務，要透過 k8s 管理
- Pod 是 k8s 中能夠被創建、調度和管理的最小單元，有自己獨立的 ip
- 一個 Pod 由一個或多個容器構成，這些容器共享 Pod 的所有資源
- K8S 分成 Master 節點和 Node 節點，可以想成實際存放 Pod 的虛擬或實體機器
- 如果 Pod 內的服務要暴露出來，就要使用 Service ，由 Service 內的 LoadBalancer 和 NodePort 提供公網 ip 和 使用的 port

---

## 安裝

我們要在本機上用 Minikube 安裝 k8s cluster ，首先要先安裝虛擬機的 hypervisor 、kubectl ，再來才是 Minikube 本身

[Minikube 的 github](https://github.com/kubernetes/minikube)上有教怎麼安裝，不過我習慣從[官網的教程](https://kubernetes.io/docs/tasks/tools/install-minikube/)上獲得完整的資訊

### 安裝 hypervisor

hypervisor 有幾種選擇，我比較熟悉~~因為免費~~的是 VirtulaBox 和 KVM，而挑選的原則也很簡單，個人用 VirtualBox 、 Server 用 KVM

之前玩 Vagrant 就已經裝過 VirtualBox 了，沒裝過的同學可以參考[在 Linux 上的安裝教學](https://www.virtualbox.org/wiki/Linux_Downloads)

補充一下快速查看 Ubuntu 版本的指令
```
    $ echo $(lsb_release -c -s)
```

然後 verify 我們裝過 VirtualBox
```
    $ vboxmanage --version
    5.0.24r108355
```

### 安裝 kubectl

最快的方式
```
    $ sudo apt-get update
    $ sudo apt-get install -y kubectl 
```

或是從最新的 stable 拉
```
    $ curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/darwin/amd64/kubectl && chmod +x ./kubectl && sudo mv ./kubectl /usr/local/bin/kubectl
```

verify kubectl 
```
    $ kubectl version
```
(是 `version` 不是 `--version`啊…)

[參考資料](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

### 安裝 minikube

```
    $ curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.23.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

[參考資料](https://github.com/kubernetes/minikube/releases)

---

## Hello MiniKube

然後我們依照 [Running Kubernetes Locally via Minikube](https://kubernetes.io/docs/getting-started-guides/minikube/) 來跑我們第一個 Minikube hello world

```
    $ minikube start
    Starting local Kubernetes v1.8.0 cluster...
    Starting VM...
    Downloading Minikube ISO
     140.01 MB / 140.01 MB [============================================] 100.00% 0s
    Getting VM IP address...
    Moving files into cluster...
    Downloading localkube binary
     148.56 MB / 148.56 MB [============================================] 100.00% 0s
    Setting up certs...
    Connecting to cluster...
    Setting up kubeconfig...
    Starting cluster components...
    Kubectl is now configured to use the cluster.
```

cluster 跑起來之後用 kubectl 控制
```
    $ kubectl run hello-minikube --image=gcr.io/google_containers/echoserver:1.4 --port=8080
    deployment "hello-minikube" created

    $ kubectl expose deployment hello-minikube --type=NodePort
    service "hello-minikube" exposed
```

deployment 根據[官方網站](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)的定義為：
> A Deployment controller provides declarative updates for Pods and ReplicaSets.

詳細 deployment 的設定怎麼寫，往後我再開一篇寫，總之上完 deployment 之後 expose 出來給外部使用

verify Pod 和 Node 的狀況
```
    $ kubectl get pod
    NAME                              READY     STATUS    RESTARTS   AGE
    hello-minikube-5bc754d4cd-vjxxc   1/1       Running   0          37s

    $ kubectl get nodes
    NAME       STATUS    ROLES     AGE       VERSION
    minikube   Ready     <none>    24m       v1.8.0

    $ kubectl cluster-info
    Kubernetes master is running at https://192.168.99.100:8443
```

然後我們可以從本機 curl 一下暴露出來的服務
```
    $ curl $(minikube service hello-minikube --url)
    CLIENT VALUES:
    client_address=172.17.0.1
    command=GET
    real path=/
    query=nil
    request_version=1.1
    request_uri=http://192.168.99.100:8080/
    
    SERVER VALUES:
    server_version=nginx: 1.10.0 - lua: 10001
    
    HEADERS RECEIVED:
    accept=*/*
    host=192.168.99.100:31843
    user-agent=curl/7.47.0
    BODY:
    -no body in request-%
```

這時候瀏覽器訪問 `client_address` 給的 `172.17.0.1` 可以看到 apache 的預設頁面
![](https://i.imgur.com/XxKUcX8.png)


也可以下 `$ minikube dashboard` 開啟瀏覽器分頁由 web console 看到 cluster 的情況
![](https://i.imgur.com/Y8a4VW9.png)


要結束的時候就壓
```
    $ minikube stop
```

覆蓋一張未完待續卡，結束這回合

