---
title: 如何用內部 mail relay 寄信 
date: 2018-02-21 14:06:35
tags:
---
<center>How to Use Internal Mail Relay Server to Send Notification</center>
===
<br>

使用機房的機器或 vm 時，常常都用送監控系統狀態的信，然後往往會使用一個擁有本地 domain 的 mail server 寄信，由於走內網，又是不怎麼具有機敏性的資訊，就不走 465, 587 port ，直接簡單用 25 port，這裡做個備忘，以下資訊包含：

- 手動使用 telnet 連結 smtp relay server 並寄信
- 在 centos 7 上安裝 mailx 並 config

---

首先 ping 看看在內網能不能碰到 relay server，以下是本篇用到的參數
- 內網 mail server: relay.mySite.com.tw
- local vm hostname: myVM.mySite.com.tw

```
    $ telnet relay.mySite.com.tw 25
    220 ESMTP Postfix

    HELO myVM.mySite.com.tw # HELO 或 EHLO 都可以，後面 hostname 亂打，對方 server 會依 dns 回正確的
    250 myVM.mySite.com.tw

    MAIL FROM:"Somebody"< somebody@mySite.com.tw > # 其實可以亂打
    250 2.1.0 Ok

    RCPT TO:"Some One Else"< someoneelse@mySite.com.tw > 
    250 2.1.0 Ok

    DATA # 輸入 DATA 準備開始寫信
    354 End data with <CR><LF>.<CR><LF>
    
    Subject: mail from tutorial
    As Title

    . # 最後一行輸入 . 並按回車鍵，信件將會立刻寄出
    250 2.0.0 Ok: queued as XXXXXXXXXX 

    QUIT
    221 2.0.0 Bye
    Connection closed by foreign host.
```

[參考資料](http://blog.faq-book.com/?p=351)

---

上面的測試過程證明了這台機器可以正確使用內網的 mail relay 寄信，接下來要用 mailx 這個套件，簡單的在 command line 寄信

- OS: CentOS 7

```
    $ sudo yum install -y mailx
    $ ln -s /bin/mailx /bin/email
    $ sudo echo "set smtp=smtp://relay.mySite.com.tw:25" >> /etc/mail.rc
    $ sudo echo "set ssl-verify=ignore" >> /etc/mail.rc
    $ sudo echo "nss-config-dir=/etc/pki/nssdb/" >> /etc/mail.rc
```

如果訊息很短的話，直接一行寄掉，加 -v 可以看到類似 telnet 的過程
```
    $ echo "Message Body" | mail -s "Message Subject" someoneelse@mySite.com.tw 
```

信稍微長一點，可以使用 redirect operator
```
    $ mail -s "Message Subject" -r '"Somebody" somebody@mySite.com.tw' someoneelse@mySite.com.tw < mail_content.txt
```

[參考資料](https://gist.github.com/ilkereroglu/aa6c868153d1c5d57cd8)

---

大致上就這樣，如果要使用外部的 smtp relay 的話，例如說 smtp.gmail.com，有使用 SSL/TLS 加密的話，就沒辦法這麼方便，可能要把帳密寫在環境變數帶入帳號驗證，使用場景不一樣，這邊就先不寫了

