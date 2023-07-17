---
title: "开始一个hugo blog"
date: 2023-05-19T16:52:59+08:00
draft: true
---

被hexo blog折磨的死去活来，干脆换一个！

于是从安装开始，首先要一个go环境，我的环境是go1.15，要升级成go1.18先：

```
 go env -w GO111MODULE=on
 go get golang.org/dl/go1.18@latest
 go1.18 download
```
go1.18环境ok了之后要用这个下载hugo：

```
go1.18 install github.com/gohugoio/hugo@latest
```

接下来就是按照hugo的文档来操作：

```
hugo new site hugoBlog
cd hugoBlog
git init
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke themes/ananke
echo "theme = 'ananke'" >> config.toml
```
最后这个echo windows下可能有问题，自己手动添加比较保险

另外默认给的这个主题看着还行，先用用。

接下来试试怎么写新文章：

```
hugo server --buildDrafts
hugo server -D