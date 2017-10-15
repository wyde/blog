---
title: How to Deploy Hexo on Github Pages
date: 2017-09-23 02:56:58
tags: 
- Github Pages
- Hexo
---

# <center>利用 Hexo 和 Github Pages 建立 blog</center>
<br>

~~第一篇文章不可免俗地來介紹這個 blog 怎麼建起來的，最陽春的那種 :)~~

結果不小心手殘誤刪，這篇無法當第一篇了，才發現 hexo deploy 只有 push public 裡 render 完的網頁，原本以為會連 markdown 都上傳，看來要用另一個 repo 管理 markdown 的 source ，難怪有人還會再接 CI(Continous Integration) 的工具，這樣寫完 markdown 就直接生成網站了，這次先簡單地寫一下架設步驟，因為是在雲端 IDE 上處理，所以不論什麼 OS 的使用者都能 remote 免費使用

主要有三個部分，

1. 設置 Node.js 的開發環境
2. 安裝與設定 Hexo 與 NexT theme
3. 編輯完推上 Github Pages

之前在考慮 Pelican 、 Hugo 、 Hexo ，聽說了 Pelican 開發較不活躍且 render 速度慢， Hugo 配置較麻煩，就採用了 Hexo ，這些只是聽說，沒有實測…

身為 Python 愛好者，希望有 Python + Jinja2 寫的靜態網站產生器同時又兼具美觀和易用性…這次架 Hexo 的感覺真的蠻快蠻方便的

---

## Node.js setup

我習慣在 linux 上工作，這邊的開發都在一個雲端 container(Ubuntu 14.04) 上工作，好處不管用 Mac 或 Windows 都可以用瀏覽器登入免費使用，網址在[這](https://c9.io/)，使用教學在[這](https://wyde.github.io/2017/10/04/Setting-up-Cloud9-Dev-Environment)，當然如果自己有習慣的工作環境只要 node 有裝起來就可以繼續裝 Hexo

--- 

## 安裝 Hexo

```
    $ sudo npm install -g hexo-cli
    $ hexo
    (列出 Hexo 常用指令，表示成功安裝)
```

我們打算拿 Github Pages 當個人的 blog，於是目錄就直接開 `<your github account>.github.io`

```
    $ cd ~
    $ mkdir <your github account>.github.io && cd <your github account>.github.io
    $ hexo init
    $ npm install
    $ hexo s -p $PORT
```
(起了 hexo 的測試 server，如果要關掉就 Ctrl + C)
另開瀏覽器分頁，可以在 `https://<c9 project name>-<c9 user name>.c9users.io`看到 Hexo 的 hello-world，如果是在本機上開發，那瀏覽器就開 `localhost` 吧，這樣 Hexo 基本上就裝完了

安裝 theme ，我選的是 NexT
```
    $ git clone https://github.com/iissnan/hexo-theme-next themes/next
    $ vim _config.yml
```
修改 theme 的設定值為 next

- 修改 Hexo 本身的設定要在 ./_config.yml
- 修改 Next 的主題設定要在 ./theme/next/_config.yml

---

## Github Pages 設定

1. 如果沒有，先申請一個 Github 帳號
2. 開一個新的 repo 叫 <your github account>.github.io
3. repo 的 settings -> Github Pages -> Source 選擇 master branch
4. 以下回到 Cloud9 console

```
    $ git config --global user.name "<github username>"
    $ git config --global user.email "<github user email>"
    $ vim ~/.ssh/<private key>
```

貼上 private key 或用 `ssh-keygen -t rsa -b 4096 -C "<your_email>@example.com"`
 產一把新的 private key，這樣其實會把 ssh private key 放在網路上的某台機器，即使這台機器必須經由密碼登入，對於安全性非常敏感的人可能不能接受私鑰放在雲端的機器上。我會生一把專用的 key ，不會用在其他工作場合的 key ，所以要採用這種方法的人請自行考慮風險～

臨時加個 ssh key ，測試成功應寫在 ~/.ssh/config
```
    $ eval "$(ssh-agent -s)"
    Agent pid 31418
    $ ssh-add ~/.ssh/<private key>
    $ ssh -T git@github.com
    Hi <your github account>! Youve successfully authenticated, but GitHub does not provide shell access.
```

先安裝 deploy 套件
```
    $ npm install hexo-deployer-git --save
```

修改 _config.yml 裡的 deploy
```
    deploy:
        type: git
        repository: git@github.com:<your github account>/<github account>.github.io.git
        branch: master
```

然後就可以測試部署
```
    $ hexo d --generate
```

開啟瀏覽器分頁，你的 blog 會在 `https://<your github account>.github.io`

---

以下是常用編修文章的指令

```
    $ cd <hexo 所在資料夾>
    $ hexo n "New Post" # 新增 post
    $ hexo s -p $PORT # 本地起測試 server
    $ hexo clean # 清除 cache，部署前清一波
    $ hexo d --generate # 部署
    
```

---

## Third Party Integration

簡單的配置就懶得寫教學了，直接參考別人的
2017-09-30 新增
- 評論系統： [Hexo 集成 Disqus 评论](http://www.cylong.com/blog/2017/03/26/hexo-next-disqus/)
- 瀏覽統計： [配置 LeanCloud](https://notes.wanghao.work/2015-10-21-%E4%B8%BANexT%E4%B8%BB%E9%A2%98%E6%B7%BB%E5%8A%A0%E6%96%87%E7%AB%A0%E9%98%85%E8%AF%BB%E9%87%8F%E7%BB%9F%E8%AE%A1%E5%8A%9F%E8%83%BD.html#%E9%85%8D%E7%BD%AELeanCloud) (如果用 Hexo  + NexT 那這篇文章上半就不用看了，直接從配置 LeanCloud 看就好)

---

## TODO

- 社群分享
- 打賞
- 模板.md

