---
title: Python Markdown Resume Generator
date: 2017-10-03 20:17:14
tags: [Markdown, Resume, Geek, 阿宅履歷器]
---

# <center>Python 阿宅履歷產生器</center>
<br>

最近剛好一直折騰 Python 、 Markdown 、 靜態網頁之間的小作品，免不了要碰 CSS ，往往這就是效能瓶頸啊，難就難在這個美感， div 擺弄半天。這跟今天的主題也有關，前兩天看了 [ptt 一篇履歷寫作相關的文章](https://www.ptt.cc/bbs/studyabroad/M.1506792069.A.449.html)，想了一下肥宅工程師如我履歷應該長怎樣，別人講了很多，直接貼連結，有興趣的就看看，不然可以拉下去直接看開發筆記

- [從上一篇文章看到的範例](https://careercup.com/resume)
- [關於 MIT 履歷的範本的報導](http://www.storm.mg/lifestyle/280401)

然後是

- [MIT 履歷範本它本人](https://gecd.mit.edu/sites/default/files/jobs/files/sample-resumes.pdf)
- [Five Steps to Writing a Great Resume](https://gecd.mit.edu/jobs-and-internships/resumes-cvs-cover-letters-and-linkedin/resumes)

大致是就是這些資料，同場加映

- [CMU 履歷範本它本人](https://www.cmu.edu/career/documents/sample-resumes-cover-letters/sample-resumes_scs.pdf)

## Survey

我的需求很簡單，我想要寫一次內容，然後可以快速套版產生清晰易懂的履歷，符合以上範本的格式。找了一些現有服務或專案

- [自動把自己 start 自己的專案作 summary](http://resume.github.io/)
- [The Markdown Resume: Markdown to HTML & PDF](https://mszep.github.io/pandoc_resume/)

這兩個感覺都是不錯的 feature ，未來可考慮 pre-build 自動版的 + 人工補齊 + 轉檔下檔，兩者的缺點都是沒有我要的版型

也有想到如果可以在線上編輯更好，像是

- [Markdown Editor](https://jbt.github.io/markdown-editor/)
- [hackmd](https://github.com/hackmdio/hackmd)
- [markdown-it](https://github.com/markdown-it/markdown-it)

## 開發筆記 

理想很豐滿，現實很骨感，我只想先符合我的需求就好，找了 python 中號稱 render markdown 最快的 [mistune](https://github.com/lepture/mistune) 就直接開擼。這種前端工作起手式都先 Cloud9 一波

…

好了，擼完了 (就這麼點 code 是能擼多久)，直接看[成果](https://wyde.github.io/md-resume-generator/)，思路很簡單，markdown 轉 HTML ，再直接用 css 修飾，懶得點直接看截圖

Markdown 長[這樣](https://raw.githubusercontent.com/wyde/md-resume-generator/master/source/resume.md)

生成 HTML 後預覽列印長這樣
![](https://i.imgur.com/rJY8K7w.png)

基本上就是 MIT 履歷範本第一個的樣式， resume.md 階層不改， css 樣式改變就可以再套別的版型 (歡迎 PR 工程人履歷的 css 版啊)，以下是我自己的 headings 階層，有興趣發 PR 補 css 的同學可以參考一下：

```
>: 地址、聯絡方式 
H1: 名字
H2: 大分項的標題，如(Education 、 Employment)
H3: 分述子項標題
h4: 分述子項副標題
* *: 地點、時間
```

## todo

- 轉 PDF
- 轉 json(machine readable)
- 生更多 css
- 線上編輯
- 自動列入 Github repo
