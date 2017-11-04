---
title: How to Install Nvidia 1080Ti GPU Driver On Fedora
date: 2017-11-03 16:54:04
tags:
- Nvidia 1080Ti GPU
- Fedora
---
<center>在 Fedora 上安裝 Nvidia 1080Ti GPU 顯卡驅動</center>
===
<br>

- Nvidia 1080TI GPU * 2 (記得要 1000 w 的電源)
- Fedora 26

過程蠻簡單的，就是把 fedora 預設的 nouveau 關掉之後灌 n 家自己的驅動，各個教學都差不多，可以參考

- [[挖礦] Nvidia 挖 Monero(XMR) on Fedora 25](https://www.ptt.cc/bbs/DigiCurrency/M.1499059041.A.ECA.html)
- [Fedora 27/26/25 nVidia Drivers Install Guide](https://www.if-not-true-then-false.com/2015/fedora-nvidia-guide/)
- [NVIDIA CUDA Installation Guide for Linux](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/)

參考別人的安裝過程中，我的原則是這樣

- 改設定檔之前先備份
- 能讓 installer 自己改設定就不手動

以下就直接開始吧

a. 裝套件

官方建議裝這些
```
    $ sudo dnf update
    $ sudo dnf install -y kernel-devel-$(uname -r) kernel-headers-$(uname -r) gcc
```

不過我這些也裝了
```
    $ sudo dnf install -y dkms acpid libglvnd-glx libglvnd-opengl libglvnd-devel pkgconfig
```

---

b. 下載驅動並關掉 nouveau
verify GPU 的資訊
```
    $ lspci | grep -i nvidia
    01:00.0 VGA compatible controller: NVIDIA Corporation GP102 [GeForce GTX 1080 Ti] (rev a1)
    01:00.1 Audio device: NVIDIA Corporation GP102 HDMI Audio Controller (rev a1)
    02:00.0 VGA compatible controller: NVIDIA Corporation GP102 [GeForce GTX 1080 Ti] (rev a1)
    02:00.1 Audio device: NVIDIA Corporation GP102 HDMI Audio Controller (rev a1)
```

顯示系統資訊
```
    $ uname -m && cat /etc/*release | head -n 1
    x86_64
    Fedora release 26 (Twenty Six)
```

依照 GPU 和系統資訊上[官方網站查詢顯卡驅動](http://www.nvidia.com/Download/Find.aspx?lang=en-us)
![](https://i.imgur.com/cg6KvgL.png)

找到該驅動下載的[說明頁面](http://www.nvidia.com/download/driverResults.aspx/125399/en-us)並下載，我裝的是 387.12 版本
```
    $ wget http://us.download.nvidia.com/XFree86/Linux-x86_64/387.12/NVIDIA-Linux-x86_64-387.12.run -P ~
    $ chmod +x ~/NVIDIA-Linux-*.run
    $ sudo ~/NVIDIA-Linux-x86_64-387.12.run
```

這邊會出現安裝程式，就一路 yes ，nvidia-installer 會建立 `/etc/modprobe.d/nvidia-installer-disable-nouveau.conf` ，把 fedora 預設的[開源顯卡驅動 nouveau ](https://zh.wikipedia.org/wiki/Nouveau)關掉

---

c. 修改 grub 開機選項

`$ sudo vim /etc/sysconfig/grub` ，找到 GRUB_CMDLINE_LINUX 這行，最行加上 `rd.driver.blacklist=nouveau`，在我機器上長這樣 
```
    GRUB_CMDLINE_LINUX="rd.lvm.lv=fedora/root rd.lvm.lv=fedora/swap rhgb quiet rd.driver.blacklist=nouveau"
```

然後根據參考資料1，
> 如果是BIOS開機，執行 $ sudo grub2-mkconfig -o /boot/grub2/grub.cfg
> 如果是UEFI開機，執行 $ sudo grub2-mkconfig -o /boot/efi/EFI/fedora/grub.cfg
> (不確定的話就看哪個檔案存在，通常不會兩個都存在)

新一點的機器應該都支援 UEFI 了

---

d. 移除 nouveau
```
    $ sudo dnf remove xorg-x11-drv-nouveau
```

---

e. 備份舊的開機映象檔，準備用新的開機
```
    $ sudo mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r)-nouveau.img
    $ sudo dracut /boot/initramfs-$(uname -r).img $(uname -r)
```

(optional) f. 設定開機為文字模式
因為我這台工作站本來就是純文字模式，如果是拿來當桌機，有進 x windows 的話，這邊先用純文字開機，裝完驅動再調回來
```
    $ sudo systemctl set-default multi-user.target
    $ sudo reboot
```

---

g. 完成安裝與檢成顯示資訊
```
    $ sudo ~/NVIDIA-Linux-x86_64-387.12.run
    $ nvidia-smi
```

如果可以看到類似的畫面，那就裝完啦，有做 f 的同學可以再調回 level 5 開機
![](https://i.imgur.com/RcKpQl8.png)

(optional) h. 有做 f. 的同學
```
    $ sudo systemctl set-default graphical.target
    $ sudo reboot
```

Done
