---
title: Jsch 初应用
date: 2021-03-04 17:25:01
tags:
- java
- jsch
categories:
- java初学者教程
---

## 什么是Jsch

Jsch是一个纯java写的ssh客户端，通过Jsch可以完全通过java来实现ssh的一些功能

## 为什么要用Jsch？

有个需求要做一个netconf长连接到设备上，监听notifaction，看了一圈好像没有现成的解决办法，所以只能自己想办法研究Jsch写一个

<!--more-->

## Jsch探索

### Jsch中连接的方式：

在Jsch中一个ssh连接称为一个Session，在一个建立好的Session中可以用 `sshSession.openChannel`获取不同类型的Channel：

|channel|用途|备注
|--|--|--|
|exec|单独执行一个命令
|shell|远程终端方式交互
|subsystem| 子系统|要用`setSubsystem`来指定子系统类型，比如cmd，netconf
|sftp|传输文件

### 获取ssh的输出

对channel有一个getInputStream的方法，可以获取到输入信息流，输出的类型是InputStream。当前的代码是这样的：

````java
InputStream inputStream = chn.getInputStream();
    try {
        //循环读取
        byte[] buffer = new byte[1024];
        int i = 0;
        //如果没有数据来，线程会一直阻塞在这个地方等待数据。
        while ((i = inputStream.read(buffer)) != -1) {
            System.out.println("got data!");
            //TODO: 将获取的代码转化为字符串
        }
    } finally {
        //断开连接后关闭会话
        deviceConn.close();
        if (inputStream != null) {
            inputStream.close();
        }
    }
````
在这里实际上每次有输入的时候，都会看到got data，但是怎么把输入流转为string做解析我还在研究怎么做···

这个[文章](https://www.baeldung.com/convert-input-stream-to-string)总结了一下如何把inputStream转化为string，但是我试了几个并不好用，最后用下列代码解决了输入流转string的问题：

````java
byte[] buffer = new byte[1024];
int i = 0;
//如果没有数据来，线程会一直阻塞在这个地方等待数据。
while ((i = inputStream.read(buffer)) != -1) {
    TextMessage textMessage = new TextMessage(buffer);
    System.out.println(textMessage.getPayload());
}
````

当然为什么这个ok也是一个坑···具体的怎么用以后慢慢填坑吧···

update 4.22：

> buffer里边存的东西是ascii，所以也可以自己转啦··这样子可能更安全