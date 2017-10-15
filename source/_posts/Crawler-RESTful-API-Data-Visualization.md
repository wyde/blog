---
title: Crawler, RESTful API & Data Visualization
date: 2017-09-19 02:05:17
tags: [鄉鎮預報, 氣象, API, 中央氣象局, 台大球場可以打嗎?, 後端Python, 前端D3, MySQL, DataScience]
---

# <center>爬蟲、RESTful API、資料視覺化</center>
<br>

TL;DR 寫了一個簡單可用的 prototype ，把抓資料、存資料、呈現資料和資料視覺化這幾件事兜起來，作為自己未來如果要作資料科學專案的腳手架，也 [open source](https://github.com/wyde/town368) 給大家鞭一鞭
[台灣 368 鄉鎮市區天氣預報 RESTful api 說明文件](https://wyde.github.io/town368)

## 背景

之前有需要用到台灣 368 鄉鎮市區的天氣預報，查了政府資料開放平台，有編號 9307 和 9309 的下列兩筆
- [鄉鎮天氣預報-台灣未來2天天氣預報](https://data.gov.tw/dataset/9307)
- [鄉鎮天氣預報-全臺灣各鄉鎮市區預報資料](https://data.gov.tw/dataset/9309)

不過都沒有提供 json 的 API ，而且內容看起來是縣市級的未來 72 小時的逐三小時的預報，不是鄉鎮市區級的，因此就寫了爬蟲抓資料，試提供 RESTful API，資料來源直接由中央氣象局的在地天氣報馬仔來抓，例如說[大安區公所](http://www.cwb.gov.tw/V7/forecast/town368/towns/6300300.htm)的範例

btw，在公部門開放資料的部目，目前看到[健康保險資料服務](http://data.nhi.gov.tw/)做得還不錯，可以參考一下，還有以下兩個規範也是未來可努力的方向
- 民104.7 國發會 [共通性資料存取應用程式介面(API)規範](http://file.data.gov.tw/opendatafile/%E5%85%B1%E9%80%9A%E6%80%A7%E8%B3%87%E6%96%99%E5%AD%98%E5%8F%96%E6%87%89%E7%94%A8%E7%A8%8B%E5%BC%8F%E4%BB%8B%E9%9D%A2API%E8%A6%8F%E7%AF%84.pdf)
- 民104.7 國發會 [資料集詮釋資料標準規範](http://file.data.gov.tw/opendatafile/%E8%B3%87%E6%96%99%E9%9B%86%E8%A9%AE%E9%87%8B%E8%B3%87%E6%96%99%E6%A8%99%E6%BA%96%E8%A6%8F%E7%AF%84.pdf)

---

## 流程與技術棧

實作一個開放資料的 API prototype，資料來源由爬蟲去抓中央氣象局的資料，存在資料庫裡，然後再做一個 RESTful API 當作前後端的接口，開放給前端的 data consumer 作應用，概念上可以參考碁峰的 <資料視覺化使用 Python 與 JavaScript> 一書，以下簡稱視覺化參考書，想做的事很像，都是為現在當紅所謂資料處理顯學，做一些基礎的工作。那本書我沒有看完，後端也用了 Python 不同的工具，但它給了我蠻多啟發，以下詳述

資料的生命週期裡
1. 我們會需要先把資料爬下來
2. 觀察格式、清理出我們要的資料
3. 存進 db ，資料持久化
4. 如果需要的話進行一些加值運算再存回 db
5. 使用 API 提供網路上存取 db 裡的資料
6. 網頁、移動裝置…etc的前端應用使用 API

前、後端是相對的，在不同語境下有其指涉對象，在這裡我指的是網路資料的提供者與使用者，依上述資料週期來看，稍微提一下參考書與我的工具選擇

### 參考書的工具選擇
1. (上述的 1、3項合併) 爬取：Scrapy + MongoDB
2. 清理、探索 / 處理：IPython + Pandas + Matplotlib
5. 提供：Flask RESTful API + MongoDB
6. 轉換：D3.js

### 我的工具選擇

稍微寫了 Scrapy ，蠻容易上手的，可是感覺比較適合簡單而且 HTML 格式清晰的網站，例如 Scrapy 官方教程的[示範網站](http://quotes.toscrape.com)，如果爬蟲需要比較複雜的互動，例如說處理 .NET 的 VIEWSTATE ，就覺得寫起來卡卡的，所以我就換一個方式，概念上我會用 pandas dataframe 或 csv 檔案當作 metadata ，適合快速迭代想法，確定存在關聯式資料庫時會大概的樣子，然後用 Jupyter + requests 來取代 Scrapy Shell 的資料調適步驟、 SQLAlchemy + MySQLdb 來取代 Scrapy Pipeline + MongoDB ，據說大量迸發的爬取可以用 multiprocessing 這個庫，不過我還沒用過

1. 抓取：Jupyter + requests
2. 清理：Jupyter + pandas + beautifulsoup
3. 持久化：Jupyter + SQLAlchemy + MySQLdb
4. 撰寫腳本：將 1-3 寫成 scripts ，如果需要計算模型或額外處理的話，在這裡連結
5. 提供：Django REST Framework + MySQLdb
6. 呈現：jQuery + D3.js + bootstrap

稍微提一下寫 Scrapy (1.4.0) 的寫法，主要修改下面檔案
- items.py (optional): 可寫可不寫，與 Django 的 models.py 概念不一樣，只是事先規範爬取的變數
- spider/myspider.py: 最主要關於抓取和 parsing HTML 的 code，如果不存在 db ，可以抓完直接 -o 輸出 json 檔
- settings.py: 各種配置，例如要不要經過 pipeline 存入 db
- pipelines.py: 爬下來的資料做後處理，連結 db 持久化

---

## 實作參考

之所以說是 prototype ，因為想法是做一個 Proof of Concept ，雖然可以用，但是缺乏安全與效能的探討，算是套套 framework 、踩踩坑，如果使用者數量有起來再來處理

跟其他的爬蟲 + API 解決方案比起來，例如說 AWS API Gateway + Lambda + Dynamodb，自己用框架可能多了一些工作，但至少使用的框架都是開源的，利於學習。還有就是我不想也不會使用 M$ 的解決方案，所以不要問說這些流程在 Windows 、.NET 下的對應關係。而且我只用到雲端資源做開發，就算 Laptop 使用 non-Linux OS，只要瀏覽器能上網也是可以玩玩看，甚至 git clone 完就可以改改 code ，做一個自己的 data processing demo。繼續往下閱讀的 prerequisite 大概就是有一點 Linux 的使用經驗，不會用 vim 的話至少要會其他 Linux 上的純文字編輯器像是 nano

### 開發環境建置

我在 [Cloud9](https://c9.io) 上的 Ubuntu14.04 環境下進行開發，然後在 CentOS 7 上部署，目前是比較偏好 redhat 系的發行版，不過雲端 IDE 好像都是 Ubuntu 居多，當然也可以自己開 CentOS 的開發虛擬機或 Container ，但就沒有雲端 IDE 方便

- [c9 開發環境建置筆記](https://wyde.github.io/2017/10/04/Setting-up-Cloud9-Dev-Environment)

還需要設置 MySQL 5.7 (待補)


### 爬蟲與資料清理使用 requests 、 pandas 、 beautifulsoup4

這邊處理兩種資料
1. (舊制)行政區域及村里代碼，對應全台 368 鄉鎮市區唯一編號，是網路上的 csv 檔案
2. 另一種是 cwb 上的網頁資料，逐三小時鄉鎮天氣預報，嵌在不是很直觀的 HTML 表格裡

直接用 Jupyter notebook 來講解
- [Lab1](https://github.com/wyde/town368/blob/master/jupyter/Lab1-fetch-data-into-pandas-dataframe.ipynb) 整理各縣市鄉鎮區行政中心區碼
- [Lab2](https://github.com/wyde/town368/blob/master/jupyter/Lab2-fetch-data-parse-with-bs4.ipynb) 抓取大安區公所的逐三小時預報
- [Lab3](https://github.com/wyde/town368/blob/master/jupyter/Lab3-create-mysql-table-with-sqlalchemy.ipynb) 利用 SQLAlchemy 創建 MySQL table
- [Lab4](https://github.com/wyde/town368/blob/master/jupyter/Lab4-insert-data-into-mysql.ipynb) 將 Lab1、Lab2 的結果利用 SQLAlchemy 存入 MySQL
- [Lab5](https://github.com/wyde/town368/blob/master/jupyter/Lab5-update-mysql.ipynb) 利用 Lab1:4 建立的流程更新 MySQL 裡的氣象資料

確定怎麼寫之後，再把 Lab1:5 寫成 [scripts](https://github.com/wyde/town368/tree/master/crawler) ，我放在 crawler 資料夾裡，方便管理與 cron job 的執行

坑： 免費版的 Cloud9 container 不支援 crontab ，用 nohup + while loop 是可以試試看啦 [參考](https://community.c9.io/t/crontab-doesnt-work-in-cloud9/4226/4)

### RESTful API 使用 Django REST Framework

[本篇範例](https://github.com/wyde/town368/tree/master/djangorest)

(教學文件待補)

### 資料視覺化使用 jQuery + D3.js

[本篇範例](http://dadacho.com)

(教學文件待補)

### Django App 部署在 CentOS 7

[本篇範例](https://town368.csie.ntu.edu.tw)

(教學文件待補)

### 說明文件使用 MkDocs

寫完一個 Project 總是要寫一些文件讓人 follow ，最簡單的方法就是 Project 的 README.md ，不過如果資料多一點的話光是 README 可能不太夠寫，就需要建立一個說明文件的簡單網站。

- 文件量少：用 README.md
- 文件量中等：文件 source 與專案一同管理
- 文件量多：文件 source 與專案分開管理，單獨部署

身為一個懶惰偏 Python 的阿宅，我只想用最少的 effort 寫前端和 javascript，寫文件這種事當然使用 static site generator ，只要寫 markdown 就可以產生還算精美的外觀，讓我可以專注在內容，如果 Google API Documentation 的話應該會找到一些測試 + 文件的生成工具，像是 Postman 、 Swagger ，可是我只是要單純告訴使用者相關的網址與參數，不需要一個互動式的測試頁面，所以我選擇了 MkDocs 來當作我寫文件的工具，內容的部分，則是參考了[這篇](https://gist.github.com/iros/3426278)

[本篇範例](https://wyde.github.io/2017/09/25/How-to-Deploy-a-Project-Site-on-Github-Pages/)
[API Documentation demo](https://wyde.github.io/town368)

---

## TODO

- db 換成 PostgresSQL
- 測試把 Django 的 ORM 換成 SQLAlchemy 的好處
- 優化爬蟲的寫法
- 用 container 開發、佈署
- 寫 test case

