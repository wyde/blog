---
title: How to Deploy a Project Site on Github Pages
date: 2017-09-25 19:13:21
tags: Github Pages
---

<center>在 Github Pages 上部署專案說明文件</center>
===
<br>

這幾天部署 [town368](https://wyde.github.io/town368) 學了不少細碎的知識，下次要用肯定會忘，只好來記個

## Project Site on Github Pages

在 Github 上部署專案的說明文件，目前有兩種方式
1. master branch
2. 在 master branch 下的 ./docs

![](https://i.imgur.com/xP6YU0o.png)

果斷選擇第二種(本 blog 則是用第一種方式發佈)，這樣在專案的主資料下可以保留說明文件的 source (markdown)檔，同時做 source 的版本管理和靜態發佈，如以下範例
```
    .
    ├── <main project>
    ├── docs
    │   └── <MkDocs 生成的靜態網站文件s>
    └── mkdocs
        ├── docs
        │   └── <你的 project 說明文件s>.md
        └── mkdocs.yml
```

(像[Hexo 一鍵部署](https://hexo.io/zh-tw/docs/deployment.html) 的 `hexo deploy` 上去就直接是 master branch ，還要另外開一個 repo 管 source...有沒有更方便的辦法還在想呢)

## 選擇 Project Documentation 系統

果斷選擇 [MkDocs](http://www.mkdocs.org/) ，可以配合非常熱門的佈景主題： readthedocs (built-in)。其它的 theme 可以參考 [community wiki](https://github.com/mkdocs/mkdocs/wiki/MkDocs-Themes) 。我個人是比較喜歡 [Material 主題](https://github.com/squidfunk/mkdocs-material)的左右欄設計，不會像 readthedocs 到大螢幕上就整個版本偏左…甚至我就是為了這個 theme 用 MkDocs 的，其它常用的文件工具還有 [Sphinx](http://www.sphinx-doc.org)

先裝 MkDocs 再裝 theme: Material
```
    $ cd <project directory>
    $ pip install mkdocs mkdocs-material
    $ mkdocs new docs && cd docs
    $ vim mkdocs.yml 
```
修改這一行 `theme: 'material'
[直接看作完的 demo](https://wyde.github.io/town368)

## 基本操作

```
    $ mkdocs serve -a $IP:$PORT # 起測試 server
    $ mkdocs build # 生成要發佈的靜態網站
```

### 新增與編輯文件

source (markdown)檔在 ./docs 裡，一開始只有 index.md ，如果要新增頁面，直接新增一個 *.md 檔，然後修改 mkdocs.yml ，例如我新增了一個 about.md ，接著在 mkdocs.yml 裡
```
    pages:
        - ['index.md', 'Home']
        - ['about.md', 'About']
```

### 坑

預設的靜態文件會產生在 ./site ，可是我們希望它 render 的時候是在 ../docs ，這樣到時候 Github Pages 才會在正確的資料夾找到我們的靜態資料，所以要修改一下 mkdocs.yml `site_dir:'../docs'`，然後就可以 `rm -rf ./site`

### 部署

MkDocs 有提供[一鍵部署](http://www.mkdocs.org/#deploying)，不過因為我把靜態資料夾改到原本專案的 ./docs 下，就直接用 master branch 管理就好，目前這樣寫小專案的文件還蠻方便的


