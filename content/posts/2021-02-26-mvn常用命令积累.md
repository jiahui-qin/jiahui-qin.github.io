---
title: mvn常用命令积累
date: 2021-02-26 15:28:50
tags:
- mvn
categories:
- 初学者教程
---

# 什么是MVN?

APACHE MAVEN 是一个项目管理和综合工具，帮助构建一个完整的生命周期框架，简化和标准化项目建设过程

一个典型的maven构建的生命周期：

|阶段|处理|描述|
|--|--|--|
|准备资源|资源复制|资源复制可以定制|
|编译|执行编译|源代码编译|
|包装|打包|创建JAR/WAR包、在pom.xml中定义提及的包|
|安装|安装|本地/远程maven仓库中安装程序包
<!--more-->
# 常用命令

## 打包跳过测试

* mvn clean install -DskipTests
    
    * 不执行测试用例，但是编译测试用例生成相应class文件到target/test-classes下

* mvn clean install -Dmaven.test.skip=true
    
    * 不执行测试用例，也不编译测试用例的类
