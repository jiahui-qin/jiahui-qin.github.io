---
title: java中父类object与子类object的转换
date: 2021-04-16 16:37:57
tags:
- java
- 语法
categories:
- java相关

---

今天写代码的时候有个小需求，需要将父类转换为子类，我也没想太多，就直接开干了，以下是示例代码：

```java
// 父类
public clas Parent {
    private String attr1;
}
//子类
public clas Kid extends Parent {
    private String attr2;
}
```
<!--more-->
接下来在一个操作子类的对象里边需要先获取到子类，再转换为父类操作，我也没想太多，就这么写了

```java
Parent parent  = new Parent();
(kid) parent.setAttr2(null)
````


刚开始感觉好像也没有问题嘛，结果运行的时候报了错：**ClassCastException**

查了以下发现java中不允许父类向子类转换，因为父类对子类并不存在is a的关系，允许以下两种情况：

* 子类转换为父类
* 子类转换为父类之后，再转换为子类

但是又要求父类转换为子类的话，要怎么办呢？一般有两种方法

* 重新生成一个子类，将父类中的值赋给子类
* 使用json来帮助转换：



    ```java
    import com.alibaba.fastjson.JSON;
    Parent parent = new Parent();
    String parentJson= JSON.toJSONString(parent);
    Kid kid =  JSON.parseObject(parentJson, Kid.class);
    ```
