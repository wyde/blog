---
title: 用 Daphne 部署 Django Channels 
date: 2017-11-24 18:14:32
tags:
- Django Channels
- bot server
- Daphne
- Nginx
- Python
- Deploy
- CentOS 7
- https
- Certbot
---
<center>Deploying Django Channels using Daphne</center>
===
<br>

看了前一陣子很紅的[小黃雞](http://simsimi.com/)，想做個網頁版的 chatbot 來玩玩，參考了[vaisaghvt 的 django-bot-server-tutorial](https://chatbotsmagazine.com/creating-a-simple-bot-server-using-python-django-and-django-channels-69fa3b775f2)還有[jacobian 的 channels-example](https://blog.heroku.com/in_deep_with_django_channels_the_future_of_real_time_apps_in_django)，寫了一個簡單的 EchoBot Server 作為原型，還在改進中，特意維持地很乾淨，以便自己將來要用 Django Channels 做 websocket 時比較容易參考，詳見 [Github Repo](https://github.com/wyde/django-channel-echobot)

這篇先來紀錄一下怎麼部署的，根據[官方文件的 Deploying 篇](https://channels.readthedocs.io/en/latest/deploying.html)，部署至少有三件事要做
- Set up a channel backend
- Run worker servers
- Run interface servers

借用 jacobian 的圖來解釋一下各個 process 之間的關係
![](https://i.imgur.com/nPJ7vGM.png)

channel backend 我用的是 redis，[官網的 getting started](https://channels.readthedocs.io/en/latest/getting-started.html)用的是 asgiref.inmemory.ChannelLayer ，不適合用在 production 的環境，不過改這個 backend ，大概在開發階段就會做了，只要改 settings.py 就好，上 production 也不需要什麼特別設定，以後講到開發再說，所以這邊要做的事其實是

- git clone 專案
- worker server 寫成 systemd service
- daphne server 另外寫成 systemd service
- 設定 nginx
- 設定 certbot for https

---

## Setup

```
    $ cd ~
    $ git@github.com:wyde/django-channel-echobot.git
    $ cd django-channel-echobot
```

- project name: bot2
- app name: echo
- (fake) address: myapp.echobot.ai

我把 Django secret key 和 ALLOWED_HOST 放在環境變數裡，可以寫在 ~/.bash_profile 之類的地方，clone 之後在 python 的虛擬環境下安裝 pip 套件，跟佈署有關的套件是 daphne 、nginx
```
    $ pip install -r requirements.txt
```

環境變數、虛擬環境，和套件裝好之後，可以開兩個 terminal 分頁來測試是不是準備好部署了
```
    (分頁1)$ ./manage.py runworker --threads <cpu core 數，大約 4、5>
    (分頁2)$ daphne -b 0.0.0.0 -p 8000 bot2.asgi:channel_layer
```

這時候瀏覽器打開 `myapp.echobot.ai:8000/chat` 應該就可以測試 echobot 的功能

---

## 設定 worker 和 daphne daemon

常見的方式有 upstart、systemd，或是用 supervisord 來寫 daemon，好像蠻多 python web app 是用 supervisord 來寫的，不過既然 CentOS 7 自帶 systemd ，我就懶得多裝一個套件了

systemd 的入門，可以參考[阮一峰的 Systemd 入门教程：实战篇](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html)， CentOS 7 上設定檔預設放在 `/etc/systemd/system/<daemon name>.service`，這邊我就直接上設定檔它本人

<script src="https://gist.github.com/wyde/dbef999aa8b9fc2f96bdc2e841349613.js"></script>

以及

<script src="https://gist.github.com/wyde/f519f92f9200e2d0523090ad7b66cff5.js"></script>

如果要用 upstart 執行 daemon ，可以參考[masnun 這篇文章](http://masnun.rocks/2016/11/02/deploying-django-channels-using-daphne/)

相關操作
```
    $ sudo systemctl start <daemon name>.service # 啟動
    $ sudo systemctl enable <daemon name>.service # 開機時啟動 
    $ sudo systemctl stop <daemon name>.service # 停止
    $ sudo systemctl disable <daemon name>.service # 取消開機啟動
    $ sudo systemctl reload <daemon name>.service # 改完設定重新載入
    $ systemctl list-units # 可搭配 管線和 grep 來查 daemon
```

坑：daphne 是用來 serve websocket 的 ASGI interface server，其它 normal Django view 的 WSGI 也可以使用 gunicorn 或 uWSGI 來 serve ，不過小專案先不考慮，都讓 daphne 來處理就好，可參考 [daphne 的 github repo](https://github.com/django/daphne/)

---

## 設定 Nginx 與 certbot

```
    $ sudo yum install -y nginx
```

接下來就是比較一般的 Django + Nginx + https 佈署套路，不過我在`proxy_pass` 那裡踩了坑，還是直接貼個設定檔，檔案位置放在 `/etc/nginx/conf.d/<設定檔名>.conf`

<script src="https://gist.github.com/wyde/1316d4a1154e42fb0d5f95b27854d5d0.js"></script>

其中 42,51 行是 certbot 自己產生的，必須要先上半部，有 `server_name` ， certbot 才能自動產生剩下的設定檔，certbot 基本上讓 lets encrpyt 這件事簡化到一個很舒服的境界，大推，其他發行版的 certbot 套件下載，可以參考[certbot 的官方網站](https://certbot.eff.org/)，我自己寫這篇的時候參考了 [Digitalocean 的教學](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-centos-7)

```
    $ sudo yum install -y epel-release
    $ sudo yum install -y certbot-nginx
    $ sudo certbot --nginx -d myapp.echobot.ai
    (照著 prompt 安裝)
```

相關操作
```
    $ nginx -t # 去檢查 /etc/nginx 底下的設定檔是否正常
    $ sudo systemctl reload nginx
```

坑
- 主設定檔的部分，`/etc/nginx/nginx.conf` 最上面的 user 要改成執行 web process 的 owner ，例如我是用 webuser 這個身份
- 第 7 行的 `server 0.0.0.0:8000;` 要搭配 daphne systemd 設定檔的第 11 行 `-b 0.0.0.0 -p 8000` 來用
- 原本在 gunicorn 的時候，我會用 socket 跟 nginx 溝通，這邊不太確定能不能用類似的作法，至少目前的作法能動，沒測效能就是了

其實坑還不少 orz，最後，設定 certbox 每日自動檢查憑證更新
```
    $ sudo crontab -e
    17 2 * * * /usr/bin/certbot renew --quiet # 代表每天早上兩點十七分執行
```

## 後記

為了這事第一次回了 stackoverflow ，然後被一個來自 Dublin City University 的正妹(？) Clíodhna 改了文法，明明就只是開頭有沒有大小寫 XDDDDD

紀念一下，[連結在此](https://stackoverflow.com/review/suggested-edits/18040411)
