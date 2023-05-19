---
title: AutoCloseable的妙用
date: 2022-04-07 16:46:45
tags:
- java
categories:
- develop
---

讲真java里有不少奇妙的接口/注解，我觉得其实都挺好用，然而日常就是用的太少了。

为啥要提到`AutoCloseable`这个接口，是因为前段时间遇到一个bug：发现很多设备连接在get之后没有正确关闭，这个问题很明显，这是代码的问题嘛，不管get过程中有没有抛出错误，都应该关闭掉这个连接。按照正常的写法，我们一般会这么写：

<!--more-->

```java
try{
    device.connect()
    Object obj = device.get()
}catch(Exception e){
    log.warn(e.get())
}finally{
    device.close()
}
```
而我犯得错误就是把`device.close()`写到了`try`里边，没有最后这个`finally`来关闭连接，导致`get`报错的时候连接没有正确关闭。

修bug的时候看了一下`device`这个类，看到他实现了`AutoCloseable`这个接口，就顺手搜了一下，发现这个还是有点意思:

1. 实现了`AutoCloseable`和`Closeable`接口的对象可以用在`try-with-resources`语句中来实现资源的自动关闭
1. 适用范围（资源的定义）： 任何实现 `java.lang.AutoCloseable`或者 `java.io.Closeable` 的对象
2. 关闭资源和 finally 块的执行顺序： 在 `try-with-resources` 语句中，任何 `catch` 或 `finally` 块在声明的资源关闭后运行

直接上代码：

```java
try(Device device = new Device()){
    Object obj = device.get()
}catch(Exception e){
    log.warn(e.get())
}
```

对比上面的写法可以说是简单了很多，就算是有多个资源要开启，也很方便:

```java
try(Device device = new Device();
    Device device1 = new Device()){
    Object obj = device.get()
    Object obj1 = device1.get()
}catch(Exception e){
    log.warn(e.get())
}
```

总之这个语法糖相当好用嘛。

