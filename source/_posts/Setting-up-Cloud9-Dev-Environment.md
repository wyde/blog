---
title: Setting up Cloud9 Dev Environment
date: 2017-10-04 05:53:59
tags: [c9, setup, 開發環境, 起手式]
---

## <center>配置 Cloud9 開發環境</center>
<br>

好像是之前在看 [freecodecamp](https://www.freecodecamp.org/) 的時候知道了 [Cloud9](https://c9.io) ，很久沒碰 freecodecamp ，但 Cloud9 就一直用到現在。用 freecodecamp 的經驗值得再書一篇我心目中的程式教育應該長怎樣，不過…以後再說吧

這篇紀錄一下我怎麼折騰 Cloud9 的開發環境，之前在[安裝 Hexo 那篇](https://wyde.github.io/2017/09/23/How-to-Deploy-Hexo-on-Github-Pages/#more)有說過， c9 基本上可以視為免費雲端的 Ubuntu 14.04 container + IDE ，當然身為 vim 黨不需要 IDE ，而且我的瀏覽器通常都會裝 cVim ，用瀏覽器寫扣快捷鍵常常會被抓走，不過開發個人的 web 小 project 還算夠用。在辦公室、在家裡一開瀏覽器就可以接上原本的 session 也很方便。最大的優點是如果要嘗試我的 tutorial 可以馬上使用一模一樣的環境，而且是免費的。最大的缺點是我不會用 Ubuntu 14.04 當作 production 的部署環境

先寫一波手動的程序，有時間再來自動化

## 先更新系統和時區

```
    $ sudo apt-get update
    $ sudo dpkg-reconfigure tzdata # (跟著 menu 選時區)
    
    $ cat /etc/timezone
    Asia/Taipei
```

## 切換使用者

不知道為什麼 c9 的預設配置會 chroot 給使用者 ubuntu ， bash 卻顯示 c9 的 user name ，`$ id`一下還蠻不解的，於是我都會先新創使用者 webuser ，以利一致性與部署

```
    $ sudo adduser webuser
    (跟著 prompt 輸入兩波新密碼)

    $ sudo gpasswd -a webuser sudo # 加入 sudo 群組
```

然後新使用者的 .bashrc 和 .vimrc 都蠻難用的，雖然平常工作機我會用 zsh + oh my zsh ，不過 c9 空間有限(看看右上角的 meter)，所以這邊我會給可工作的最小配置

```
    $ sudo wget https://cdn.rawgit.com/wyde/467c9d52b5f87c8914cb3e2b5e776e2d/raw/a2655490772f21c8e656391577d2ee6d5b878f08/gistfile1.txt -O /home/webuser/.bashrc; \
    sudo wget https://cdn.rawgit.com/wyde/1a0623616218ccfa04579ba5608e61ff/raw/9f247244197c3eb0a02c7d04f5be81ffebc92e87/gistfile1.txt -O /home/webuser/.vimrc
```

這裡 .vimrc 只設定行號和四格空白的縮進，設定好了就可以開始使用新的 user 身份
```
    $ su webuser
    $ cd ~
```

## 設定 ssh key

這裡就比較見仁見智了，因為要把 private key 放在這個 container 裡，多少有點風險，自行斟酌吧，我是生了一對 c9 和 github 專用的 key ，掉了就算了，個人 side project 其他地方也有備份

```
    $ mkdir ~/.ssh && chmod 700 ~/.ssh
    $ touch ~/.ssh/<private key> && chmod 600 ~/.ssh/<private key>
    $ vim ~/.ssh/<private key> # 直接貼上 private key
```

(如果要新增 key pair)
```
    $ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

然後把 key 加入 ssh-agent 測試一波
```
    $ eval "$(ssh-agent -s)"
    $ ssh-add ~/.ssh/<private key>
    $ ssh -T git@github.com
    Hi <github username>, Youve successfully authenticated, but GitHub does not provide shell access.
```
看到這行就算加完了

以上是 general 的設定，接下來根據不同語言、工具的開發需求稍微紀錄設定方式

## Python 3 (3.4)

預設的 python 還是 2.7.6 ，我打算升到 3 ，然後在 virtualenv 裡面開發，然後懶得用 wrapper 了，覺得 container 裡面基本只會執行一個單純的 project ，就不用多設定一層

```
    $ sudo pip install pip --upgrade
    $ sudo pip install virtualenv
    $ virtualenv -p python3 ~/.env # 還是放在隱藏目錄下順手
    $ source ~/.env/bin/activate
    (.env) $ python --version
    Python 3.4.3
```

以後 tutorial 就懶得多寫一個 (.env) 了，執行 python project 的基本就不再特別標註

## Node.js (v6.11.3)

自從從 freecodecamp drop out 就沒在用 node 開發東西了，這裡的設置主要是為了 Hexo ，不過在這大前端時代感覺就會玩到，至少 server side 我是不會想用啦

```
    $ cd /tmp
    $ curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
    $ sudo apt-get install -y nodejs
    $ node --version
    v6.11.3

    $ cd -
```

---

(2017-10-16)
## initial push to Github

### licence

可以直接從網頁介面選擇：參考 [Choose an open source license](https://choosealicense.com/)

### 新增 README.md 的 template
```
    $ cd <project directory>
    $ wget https://cdn.rawgit.com/wyde/e951a99f6eec2cbeae99bbd9b530873d/raw/83bc4294cafe5a74c977fbf9c27acb2fbe390570/readme-template.md -O README.md
```

### initial push
```
    $ cd <project directory>
    $ git init
    $ git commit -m "initial commit"
    $ git remote add origin git@github.com:<github account>/<repo name>.git
    $ git push -u origin master
```

## TODO

- 設置 Jupyter
- setup MySQL
- setup Redis
- setup PostgreSQL
