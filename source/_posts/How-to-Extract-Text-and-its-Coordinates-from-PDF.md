---
title: How to Extract Text and its Coordinates from PDF
tags:
  - PDFminer
  - extract PDF
  - text coordinate
date: 2017-11-03 03:14:45
---

<center>如何取出 PDF 裡的文字與其座標</center>
===
<br>

今天遇到一個小任務，要把一票 PDF 特定區域的文字抓下來，有幾個剛需
- 區域沒有統一的欄位名稱
- 缺乏可識別特徵只知道是英數字組合
- 那些 PDF 有好幾個版型，希望一開始能用人工做一個版，後續批次套用
- 該處的文字應該還是標準的 text 非手寫

…嗯，說真的是難以理解的需求，我一開始的想法是這樣：

1. PDF 轉圖片用 OpenCV 手動標記做 mask
2. 截取一小塊來 OCR
3. 文字輸出， done

代誌毋係憨人所想欸亞你乾擔，然後某生統強者說有一個套件叫 PDFMiner，可以拿到他需要的 text 的 location ，就來試用一下

## 環境與安裝

- OS: Fedora 26
- Python 3.5 以下 (這很重要，一開始裝到 3.6 的 virtualenv 浪費很多時間)

首先建立 virtual environment ，可依自己的習慣或是我[之前的筆記](https://wyde.github.io/2017/10/30/Setting-up-Python-3-Virtual-Environment-on-CentOS-7/)，理論上 python 2.7 的套件也可適用，不過我會用 python 3 的語法

```
    $ pip install pdfminer.six
```

## 測試

上網隨便找一個簡單的 PDF 文件測試，裝完 PDFminer 之後，根據[官方文件](https://media.readthedocs.org/pdf/pdfminer-docs/latest/pdfminer-docs.pdf)，有兩支 scripts 可以用
- pdf2text.py
    > pdf2txt.py extracts text contents from a PDF file.
- dumppdf.py
    > dumppdf.py dumps the internal contents of a PDF file in pseudo-XML format. This program is primarily for
debugging purposes, but it’s also possible to extract some meaningful contents (such as images).

不過這邊有個小坑就是 scripts 的開頭是 `#!/usr/bin/env python` ，不是我們虛擬環境的 python ，所以直接拿掉這行，下面直接改用 ~/.env 裡的 python 來執行

```
    $ wget http://unec.edu.az/application/uploads/2014/12/pdf-sample.pdf -P .
    $ python ~/.env/bin/pdf2txt.py pdf-sample.pdf -o pdf-sample.html
```

然後用文字編輯器或瀏覽器都可以打開 `pdf-sample.html` 看看

---

## 開工

我們的目標是列出每個文字方塊(LTTextBoxHorizontal) 的座標參數與文字內容， code 沒幾行，主要也是參考官方文件和[這篇討論](https://stackoverflow.com/questions/31819862/python-pdf-mining-get-position-of-text-on-every-line)，不過官方文件的範例 code 要斟酌使用， `PDFResourceManager` 應該是在  `pdf.miner.pdfinterp` 裡，不是在 `pdfminer.converter` 裡

```=python
from pdfminer.layout import LAParams
from pdfminer.converter import PDFPageAggregator
from pdfminer.pdfinterp import PDFResourceManager, PDFPageInterpreter
from pdfminer.pdfpage import PDFPage
from pdfminer.layout import LTTextBoxHorizontal

document = open('pdf-sample.pdf', 'rb')
rsrcmgr = PDFResourceManager()
laparams = LAParams()
device = PDFPageAggregator(rsrcmgr, laparams=laparams)
interpreter = PDFPageInterpreter(rsrcmgr, device)

for page in PDFPage.get_pages(document):
    interpreter.process_page(page)
    layout = device.get_result()
    for element in layout:
        if isinstance(element, LTTextBoxHorizontal):
            obj = element._objs[0]
            print("x_cor: %.2f " % obj.bbox[0])
            print("y_cor: %.2f" % obj.bbox[1])
            print("length: %.2f" % obj.bbox[2])
            print("height: %.2f" % obj.bbox[3])
            print("text: ", obj.get_text().replace('\n',''))
            print("--------------------")
```

執行的輸出大致長這樣
```
$ python convert.py 
x_cor: 213.76 
y_cor: 754.08
length: 381.90
height: 774.07
text:  Adobe Acrobat PDF Files
--------------------
x_cor: 89.92 
y_cor: 726.73
length: 505.64
height: 742.50
text:  Adobe® Portable Document Format (PDF) is a universal file format that preserves all
--------------------
```

可以對照[原來的 PDF 範本](http://unec.edu.az/application/uploads/2014/12/pdf-sample.pdf)來看結果，PDFminer 還有許多強大的功能，除了可以取得 text 座標與內容，圖片的座標也可以，有機會再來試試

