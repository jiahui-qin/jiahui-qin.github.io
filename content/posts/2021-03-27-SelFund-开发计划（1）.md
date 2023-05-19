---
title: SelFund 开发计划（1）
date: 2021-03-27 15:00:49
tags:
- selFund
categories:
- 开发计划
---

终于开始计划开发了，第一步先选定框架，和搭建环境

## 框架

原来计划用java，无奈本地没有装java环境，而正好有go的环境，就直接选了go里边star最高的项目： gin

https://github.com/gin-gonic/gin

数据库在选择在ubuntu里边装docker，里边再加一个mysql好了。

## 搭建环境

go的环境搭建就不讲了，这里主要写一下其他方面的东西：


### gin&开发相关：

先确保本地环境搞过goproxy代理，然后直接 

    go get -u github.com/gin-gonic/gin

接下来按照quick start里边开始，新建一个example.go，运行的时候如果报错呢，可以按照如下操作

    go mod init gin
    go mod edit -require github.com/gin-gonic/gin@latest

这个时候就会成功run起来，可以请求一下 *localhost:8080/pin*,就可以看到结果。

如果想要热启动的话，可以参照一下[fresh](https://github.com/gravityblast/fresh)项目：

    go get github.com/pilu/fresh
    cd project
    fresh

### docker相关

在ubuntu上安装docker：

    curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
    service docker start
    docker pull mysql
    docker run -itd --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql

接下来就可以用navicat来操作数据库了！

先来建表吧！

首先要存的就是基金的持仓信息*hold_share_info*：

    CREATE DATABASE selfund
    USE selfund
    CREATE TABLE hold_share_info

        DROP TABLE IF EXISTS `hold_share_info`;
    CREATE TABLE `hold_share_info`  (
      `id` int NOT NULL AUTO_INCREMENT,
      `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
      `hold_list` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL,
      `stock` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL,
      `bond` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL,
      `cash` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL,
      `total` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL,
      `update_time` datetime NULL DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP,
      PRIMARY KEY (`id`) USING BTREE
    ) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_0900_ai_ci ROW_FORMAT = Dynamic;
    
    SET FOREIGN_KEY_CHECKS = 1;




### 数据来源

[小熊同学](https://www.doctorxiong.club/)

