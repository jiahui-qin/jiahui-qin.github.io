---
title: GKD! hexo + github pages + next 踩坑过程
date: 2021-02-18 17:17:24
tags:
- github
- hexo
- blog
categories:
- 指南
---

其实在年前就试着想搞这么一套个人blog，然而年前只想着放假并没有开搞，于是在年后按照教程搞了一遍。

----
先讲一下总体思路：

github pages 可以展示静态页面

hexo 作为一个blog系统可以生成静态页面

next 其实就是一个主题，用来凑数的

Travis CI 来做集成

<!--more-->
-----

接下来是正文：

1. 本地环境搭建，生成本地blog

    按照hexo需要两个软件：Node.js 和 git，这两个的安装不在赘述

    安装hexo：

        npm install -g hexo-cli 

    hexo安装完成之后，使用以下命令可以新建一个hexo博客：

        hexo init <floder>

    floder为文件夹的名称，也就是新建的博客名，进入到这个文件夹中 *_config.yml* 是配置文件

    到了这里本地的blog也就建好了，下一步我们将其迁移至github上

2. 迁移至github

    这里直接讲将站点文件公开的部署方法：

    1. 在github上新建一个repository，这个repository的命名按照<项目名>.github.io的形式来命名（大部分人的这里的项目名都是直接用的github的用户名，可以直接用<用户名>.github.io的形式来访问这个博客。）

    2. 将github上的repository clone至本地（一般建好的情况下会有一个gh-pages的分支有一些example页面文件，先不要管他），在本地新建一个master分支，将本地blog文件夹里的所有文件copy到master分支下。

    3. travis CI配置：
    
        1). 将[travis CI](https://github.com/marketplace/travis-ci)配置到自己的github账户中

        2). 打开github的[应用设置](https://github.com/settings/installations),里边选择travis ci后边的config，将其权限配置为运行访问所有repository

        3). 在github中新建一个[token](https://github.com/settings/tokens),并勾选token的 repo 权限， 记录生成的token

        4). 在[travis CI](https://travis-ci.com/)的页面，打开刚刚建立好的github项目的配置页面，在environment variables 下新建一个变量，变量名为 *GH_TOKEN*， 变量值为刚刚github上生成的token， 点击add保存

    4. 修改travis ci配置

        在刚刚复制过去的文件夹里，新建一个./.travis.yml文件

            sudo: false
            language: node_js
            node_js:
              - 10 # use nodejs v10 LTS
            cache: npm
            branches:
              only:
                - master # build master branch only
            script:
              - hexo generate # generate static files
            deploy:
              provider: pages
              skip-cleanup: true
              github-token: $GH_TOKEN
              keep-history: true
              on:
                branch: master
              local-dir: public
    5. push master branch

        到了这一步就基本完成了，将上述文件推送至github上，travis ci就会自动检测到此项目有变化，部署博客文件至gh-pages分支上。

    6. 访问网站

        如果项目名是github用户名的话，可以直接访问 https://<你的 GitHub 用户名>.github.io

        如果不是的话，就访问 https://<你的 GitHub 用户名>.github.io/<项目名>

-----

坑：

1. 如果项目名不是github用户名的话，github还是会访问https://<你的 GitHub 用户名>.github.io 下加载css文件，这个时候需要到 _config.yml 文件下修改url为https://<你的 GitHub 用户名>.github.io/<项目名>,以及修改root为/<项目名>/，才可以正常读取到css文件。

    比如我的URL设置就是如下：

        # URL
        ## If your site is put in a subdirectory, set url as 'http://example.com/   child'     and root as '/child/'
        url: https://jiahui-qin.github.io/dragonFlyInSky.GitHub.io/
        root: /dragonFlyInSky.GitHub.io/
        permalink: :year/:month/:day/:title/
        permalink_defaults:
        pretty_urls:
          trailing_index: true # Set to false to remove trailing 'index.html'     from    permalinks
          trailing_html: true # Set to false to remove trailing '.html' from    permalinks


----

参考：

[hexo官方文档](https://hexo.io/zh-cn/docs/)

----
ps

写完之后才发现这样的教程也太多了。
