---
title: How to Install VM on Linux KVM Virtualization Host using Kickstart File
date: 2017-10-15 20:09:16
tags:
- KVM
- CentOS 7
- Kickstart
- virt-install
- vmhost
- virtual machine
- system administration
- ta217
- 虛擬機
- 基礎建設
---

<center>在 Linux KVM 虛擬化平台上自動安裝虛擬機</center>
===
<br>

目標讀者：有幾台機器需要跑服務又沒什麼預算的網管、對套裝軟體或圖形化介面感到排斥的 CentOS 主機管理者

這篇算是[上一篇](https://wyde.github.io/2017/10/13/How-to-Mount-a-Second-Drive-Managed-by-LVM/)的延伸，實體機 RAM 和硬碟插滿滿的，當然就要開 VM 來跑服務啦，除非是效能考慮必須用整台實體機的資源。當初自學虛擬化平台也是一個大坑啊… x86 上的 VMWare、VirtualBox、KVM 都碰過了， Xen 還沒玩過…感想是，如果在個人電腦上要開虛擬機做實驗，推薦 VirtualBox + Vagrant ，安裝方便又有 command line 可以使用，不過現在 Vagrant 好像已經不紅了… server 上推薦使用 KVM ，免費開源又可以用 scripts 進行大量客製化

有關 KVM 的介紹我懶得打了，可參考[這篇](http://www.lijyyh.com/2015/12/linux-kvm-set-up-linux-kvm.html)，我只會紀錄實作筆記。使用 KVM 在沒有圖形化介面的 server 上開 VM 有很多種方法，我只列幾種

1. virt manager，例如[鳥哥的教學](http://linux.vbird.org/linux_basic/0157installcentos7.php)
2. virt-install with graphic ，再跑一個 vnc server 
3. virt-install 用文字的交互式介面輸入參數
4. virt-clone 另一個現在的 domain 再進該 VM 改參數
5. virt-install 讀 VM 的 xml 設定檔
6. virt-install with kickstart file

基本上如果要大量地開、重複地開、自動化地開、按照心中定製的樣子的開，選項 1-4 就要先刪掉了，選項 5 我沒試過， xml 讀起來眼睛很花，所以我使用選項 6 的方式。網路上找到用 virt-install 的教學大多是 2 或 3，甚至很多 scripts 我都懷疑有沒有測試過…以下 scripts 都是測過真的能用的

## 實驗環境

- Virtualization Host：CentOS 7
- Guest VM：CentOS 7 (redhat 系的理論上都可以用 kickstart)
- 目錄結構：
    - `/image/iso` 放在 CentOS 7 Everything 的 iso 檔，可從[這裡](http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Everything-1708.iso)下載
    - `/image/qcow2` 下掛放 qcow2 映象檔
    - `/home/ops/vmhost-manager` 放管理用的 script 和 kickstart file

## 流程

1. Virtualization Host 的硬體設定和套件安裝
2. 挖一塊 qcow2 的 image
3. 整理 kickstart file ，輸入網路…等的參數
4. virt-install

### vmhost 設定和套件安裝

快速安裝套件
```
    $ sudo yum install -y qemu-kvm qemu-img libvirt libvirt-client virt-install bridge-utils
```

1. 先確定 cpu 有支援虛擬化，`$ grep -E '(vmx|svm)' /proc/cpuinfo` 看有沒有出現 vmx 或  svm 的字，也可以下 `$ sudo virt-host-validate` 檢查是否虛擬化 ready ，準備工作可以參考[這篇](https://godleon.github.io/blog/2016/07/27/QEMU-KVM-In-CentOS7-GettingStart)
2. 在安裝 vmhost 的 OS 時， software selection 就要選擇 Virtualization Host -> Virtualization Platform ，可以安裝與設定環境的很多麻煩
3. 確定虛擬機映象檔要掛的目錄下有足夠的空間，本篇範例是掛在 `/image` 下
4. 設定 network 的 bridge 模式，因為我們的虛擬機要自己拿 ip 對外連線，或至少要在內網能與其它機器互相溝通，就不能選擇 NAT 或 isolate 的模式，這邊可以手動設定 `/etc/sysconfig/network-scripts/ifcfg-<xxx>` 把原來 host 機器上的 interface 橋接出來，或是簡單一點，使用 `$ sudo nmtui` 以 NetworkManager 來進行設定，如果沒有 bridge 選項，先查看 `bridge-utils` 安裝了沒

### qemu-img

qcow2 的格式可以看[IBM 的介紹](https://www.ibm.com/developerworks/cn/linux/1409_qiaoly_qemuimgages/index.html)，這邊開了一個 50G 的 qcow2 文件
```
    $ sudo qemu-img create -f qcow2 /image/qcow2/vm-testing.qcow2 50G
```

### kickstart file

雖然很長，不過還蠻重要，就全部用 gist 的連結貼上來了，每個安裝完的 CentOS 7 在 `/root/anaconda-ks.cfg` 都會有這台機器安裝時的 kickstart 範例，蠻有參考性的，下面的範例要注意幾個地方
- disk 的 device name 預設是 vda 不是 sda，用 sda 會報錯
- network 設了兩個 interface ，對應到 host 上面要有兩個相同網段的 bridge，這邊的數量依需求調整
    - guest vm eth0 <-> host os br1
    - guest vm eth1 <-> host os br2
- 我不喜歡寫太複雜的 kickstart file ，雖然他功能很強大，不過我把 kickstart 這邊的角色設定在能夠把基本的 os 、 network 、 rootpw 帶起來就好，其他的 config management ，我傾向交給 ansible、puppet 一類的管理工具來處理，生起來的 os 乾乾淨淨就好

<script src="https://gist.github.com/wyde/5da0c89a81a844339119e715c9ec38b9.js"></script>

### virt-install

這邊的參數要跟前兩步配合
- disk size 要跟 qemu-img create 時的大小一樣
- network 的數量要跟 kickstart 合得起來

virt-install 有些參數功能跟 kickstart file 是 overlap 的，例如兩者都可以設 ip ，我自己的原則是網路的部分給 kickstart 設，virt-install 的參數越少越好，不然很長…可以參考以下範例自己寫 scripts

要特別注意
- `--initrd-inject` 和 `--extrs-args` 裡 ks file 的寫法，前者要寫完整的路徑(和檔名)，後者只要寫檔名
- 用`--extra-args` 之後，如果 iso 位置前面的參數是 `--cdrom` 的話會報錯，必須用 `--location`

```
    NAME=vm-testing
    KSNAME=$NAME.cfg
    DISKSIZE=50G 
    
    virt-install \
        --name $NAME \
        --vcpus 1 \
        --ram 1024 \
        --disk path=/image/qcow2/$NAME.qcow2 \
        --disk size=DISKSIZE \
        --os-variant rhel7 \
        --network=bridge:br1 \
        --network=bridge:br2 \
        --nographics \
        --initrd-inject /home/ops/hostmanager/kickstart/$KSNAME \
        --extra-args "ks=file:/$KSNAME console=tty0 console=ttyS0,115200n8" \
        --location=/image/iso/CentOS-7-x86_64-Everything-1708.iso
```

## Reference

- 這邊推薦必備的參考資料就是 redhat 官方的[Red Hat Enterprise Linux 7 - Virtualization Deployment and Administration Guide ](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/pdf/Virtualization_Deployment_and_Administration_Guide/Red_Hat_Enterprise_Linux-7-Virtualization_Deployment_and_Administration_Guide-en-US.pdf) 這比網路上獅子的鬃毛要可靠多了，5xx 頁拿來當參考書
- 嫌樓上太長的也可以看 4x 頁，一樣是官方連結的 concept [Red Hat Enterprise Linux 7 - Virtualization Getting Started Guide](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/pdf/Virtualization_Getting_Started_Guide/Red_Hat_Enterprise_Linux-7-Virtualization_Getting_Started_Guide-en-US.pdf)
- 整個流程還有很多可以自動化的部分，正在開發 scripts ，有興趣可看 [github:vmhost-manager](https://github.com/wyde/vmhost-manager)
