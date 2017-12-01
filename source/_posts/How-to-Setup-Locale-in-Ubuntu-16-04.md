---
title: How to Setup Locale in Ubuntu 16.04
date: 2017-11-07 12:20:39
tags:
---
<center>在 Ubuntu 16.04 中設定 locale 環境變數</center>
===
<br>

新裝乾淨的 ubuntu 通常 locale 都沒設好，這會導致一些錯誤，像是 pip install 就會炸了
```
$ pip install virtualenvwrapper
Traceback (most recent call last):
  File "/usr/bin/pip", line 11, in <module>
    sys.exit(main())
  File "/usr/lib/python2.7/dist-packages/pip/__init__.py", line 215, in main
    locale.setlocale(locale.LC_ALL, '')
  File "/usr/lib/python2.7/locale.py", line 581, in setlocale
    return _setlocale(category, locale)
locale.Error: unsupported locale setting
```

這裡簡單備忘一下如何快速將 locale 全域設成 zh_TW.UTF-8
1. 設定要產生的語系 `/var/lib/locales/supported.d/<config name>`
2. 產生語系設定檔 `$ sudo locale-gen` 去吃第 1. 點的設定
3. `update-locale` 語系，由系統寫入全域設定 `/etc/default/local`

以下說細一點，有錯請指正

---

首先用 `$ locale` 看一下現在的語系設定
```
$ locale
locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_ALL to default locale: No such file or directory
LANG=en_US.UTF-8
LANGUAGE=
LC_CTYPE=zh_TW.UTF-8
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=
```

新增一個檔案在 `/var/lib/locales/supported.d/locale`，內容如下
```
zh_TW.UTF-8 UTF-8
en_US.UTF-8 UTF-8
```

用 `locale-gen` 產生設定檔，可用 `$ locale -a` 列出目前支援的語系設定
```
$ sudo locale-gen
Generating locales (this might take a while)...
  en_AG.UTF-8... done
  en_AU.UTF-8... done
  en_BW.UTF-8... done
  en_CA.UTF-8... done
  en_DK.UTF-8... done
  en_GB.UTF-8... done
  en_HK.UTF-8... done
  en_IE.UTF-8... done
  en_IN.UTF-8... done
  en_NG.UTF-8... done
  en_NZ.UTF-8... done
  en_PH.UTF-8... done
  en_SG.UTF-8... done
  en_US.UTF-8... done
  en_ZA.UTF-8... done
  en_ZM.UTF-8... done
  en_ZW.UTF-8... done
  zh_TW.UTF-8... done
Generation complete.
```

用 `$ sudo update-locale LC_ALL="zh_TW.UTF-8"` 將設定檔寫入， 設定 `LC_ALL` 會覆寫全部 locale 中全部 `LC_` 開頭的設定，這個指令會去寫 `/etc/default/locale`，所以也可以 cat 它確認究竟寫了什麼設定進去，當然也可以只 update 自己想要的參數，改完重新登入即可

檢查一下
```
$ locale
LANG=en_US.UTF-8
LANGUAGE=
LC_CTYPE="zh_TW.UTF-8"
LC_NUMERIC="zh_TW.UTF-8"
LC_TIME="zh_TW.UTF-8"
LC_COLLATE="zh_TW.UTF-8"
LC_MONETARY="zh_TW.UTF-8"
LC_MESSAGES="zh_TW.UTF-8"
LC_PAPER="zh_TW.UTF-8"
LC_NAME="zh_TW.UTF-8"
LC_ADDRESS="zh_TW.UTF-8"
LC_TELEPHONE="zh_TW.UTF-8"
LC_MEASUREMENT="zh_TW.UTF-8"
LC_IDENTIFICATION="zh_TW.UTF-8"
```

## 參考資料
- [[Ubuntu] 如何設定語系locale](http://www.davidpai.tw/ubuntu/2011/ubuntu-set-locale/)
