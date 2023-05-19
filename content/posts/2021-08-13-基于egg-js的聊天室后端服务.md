---
title: 基于egg.js的聊天室后端服务
date: 2021-08-13 11:28:10
tags:
- nodejs
- eggjs
- 开源
categories:
- 开源项目
---

这大概是我写的第一个开源项目：本来是看到了阿里云的服务端性能大赛，就想要去插一脚去试试，结果没想到把程序写好之后卡在了部署上边，正好也有点忙（bushi），就干脆放弃了。所以就在这里把之前写好的代码开源出来，也算是迈出了个人的一小步吧。
<!--more-->

## 主要思路

### 系统要求

这个聊天室的要求其实很简单： 也就是做一套有bearerAuth token认证的服务系统，可以完成注册账号、登录、登出、创建聊天室、进出聊天室、发言、信息查看等功能就ok。

### 技术选型

最开始的时候是在java、python、go之间徘徊：

* java是最熟悉的，但是总感觉spring这一套比较重，不太想搞一套，就放弃了java。

* python只了解一些django，看了一下新版本的好像变了不少东西··电脑也没有装python环境，索性也就放弃了。

* go实在是中文资料太少了，想要试一下gin框架感觉指导资料不太多··

正好之前维护了一个组里的nodejs项目，基于阿里的egg.js，维护的时候印象相当好：结构非常明确，有完整的中文文档，调试也非常方便，就干脆用egg.js撸了一套。

数据存储呢用的是mysql，本来想要用mysql + influxdb，感觉influxdb用来存储聊天室的聊天记录非常搭配，一个measurement就是一个聊天室的记录，后来精力有限也就没搞了。

## 系统架构

在官方给的swagger文档里边，把服务主要分了三块： room、user、message，但是实际写的时候，还是感觉给的 服务 - api的对应关系不太合理，就自己做了重新区分：

* room
    * creat new room
    * get room information
    * get roo user
    * get room list

* user
    * enter room
    * leave room
    * create new user
    * login
    * get user by name
    * send message
    * get old message

删掉了message这一块，把一些room的服务移动到了user里边，整体来看更加耦合一些。

## 数据库构建

构建数据库的时候主要在纠结几点：

1. 如何将user和他enter的chatRoom对应起来
2. 聊天信息如何存储

第一个问题就用了一个state表来存储每个user和roomid的对应关系，用userid作为主键，也可以保证每个user只能进入一个room。

第二个问题当前采用的方案是所有的聊天记录都放在一个record表里边，这样确实效率有点低了。我的理想方案是用influxdb来做存储，出于聊天记录持久化的目的，保存策略应该设置为不过期，然后每个roomid都是一个measurement，对应的字段就只有userid和发言内容，时间也可以直接用influxdb自动生成的时间，这样感觉起来就简介了很多。

## 知识上的薄弱点

对于代理、并发提交这块没研究过，有时间是要学习一下
## 代码地址

接下来上我的github地址：

[nodeChatRoom](https://github.com/jiahui-qin/nodeChatRoom)