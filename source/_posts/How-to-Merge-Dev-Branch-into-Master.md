---
title: Git 小劇場：如何合併開發分支到主分支 
date: 2017-11-22 17:00:08
tags:
---
<center>How to Merge Dev Branch into Master</center>
===
<br>

最近逐漸在自己開發的 side project 也用上 git ，把 .bak 的習慣改掉
- 在 dev branch 上開發， rebase 回 master 所在的 commit
- (在 public repo 上 repo 要用 rebase 的話要注意跟別人 code 的 conflict) 
- checkout 回 master，merge
- push master to remote repo
- 刪除 local dev branch

---

## memo

```
(on dev branch) 
$ git lg
* 13a704f (HEAD, dev) plaintext echo bot prototype done
* cb876e8 (master) initail commit

(on dev branch)
$ git rebase master
Current branch dev is up to date.
```

這篇範例是比較簡單的狀況，dev 的根本來就是 master

```
(on dev branch)
$ git checkout master
Switched to branch 'master'

(on master branch)
$ git lg
* 13a704f (dev) plaintext echo bot prototype done
* cb876e8 (HEAD, master) initail commit

(on master branch)
$ git merge dev
Updating cb876e8..13a704f
Fast-forward

(on master branch)
$ git lg
* 13a704f (HEAD, master, dev) plaintext echo bot prototype done
* cb876e8 initail commit

(on master branch)
$ git branch
  dev
* master

(on master branch)
$ git branch -d dev
Deleted branch dev (was 13a704f).

(on master branch)
$ git branch
* master
```

---

## 注意的點
- `$ git merge` or `$ git rebase`
    - public: merge
        - 理由為保留紀錄
    - private or not affect other: rebase 
        - 理由為線性、乾淨
- `$ git merge` or `$ git merge --no-ff`

---

## Ref
- [zlargon - 建立 / 刪除分支](https://zlargon.gitbooks.io/git-tutorial/content/branch/create_delete.html)
- [stackoverflow(2013) - Merge development branch with master](https://stackoverflow.com/questions/14168677/merge-development-branch-with-master)
- [代码合并：Merge、Rebase 的选择(原 Atlassian 文章)](https://github.com/geeeeeeeeek/git-recipes/wiki/5.1-%E4%BB%A3%E7%A0%81%E5%90%88%E5%B9%B6%EF%BC%9AMerge%E3%80%81Rebase-%E7%9A%84%E9%80%89%E6%8B%A9)
- [連猴子都能懂的 Git 入門指南 - 分支的合併](https://backlog.com/git-tutorial/tw/stepup/stepup1_4.html) 
