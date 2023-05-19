---
title: MyBatis-Plus 使用tips
date: 2021-11-26 23:51:45
tags:
- MyBatis-Plus
- Java
categories:
- backend
---

这两天刷知乎的时候，发现有这么一个问题：“你为什么不用mybtais-plus”，大致进去翻了一下发现自己使用mybatis-plus的姿势还有很多不正确的地方，于是打算再翻一遍mybatis-plus的介绍，想一下有什么东西是可以优化的。

<!--more-->

1. 最明显也是最该优化的地方就是用`LambdaQueryWrapper`来代替`QueryWrapper`

    原先不会的时候，是用的这种写法：

    ```java
    QueryWrapper<Book> query = new QueryWrapper<>();
    query.eq("id",123)
    bookService.select(query)
    ```

    写的时候就发现了明显的问题：如果有做很多查询的话，就要硬编码很多个数据库的属性放置在service或者map里边，这样显然不安全也不优雅。所以就可以使用`LambdaQueryWrapper`来代替：

    ```java
    LambdaQueryWrapper<Book> query = new LambdaQueryWrapper<>();
    query.eq(Book::getId, 123);
    bookService.selectList(query);
    ```
    这样写显然要比用`queryWrapper`来优雅的很多，也安全很多。

2. MBP可以选择排序字段，看文档的话就可以看到：`@OrderBy`注解可以指定sql排序

    

    |属性	|类型	|必须指定	|默认值	|描述|
    |--|--|--|--|--|
    |isDesc|	boolean|	否|	是|	是否倒序查询|
    |sort|	short|	否|	Short.MAX_VALUE|	数字越小越靠前|

3. 动态表名，大概看了一下，感觉这个功能对于读取纵向分割的表还是很有用的，可以手动指定对应的表名，比如按照时间来做表的划分。看一下gitee里的这个[例子](https://gitee.com/baomidou/mybatis-plus-samples/blob/master/mybatis-plus-sample-dynamic-tablename/src/main/java/com/baomidou/mybatisplus/samples/dytablename/config/MybatisPlusConfig.java)大概就很清楚了

