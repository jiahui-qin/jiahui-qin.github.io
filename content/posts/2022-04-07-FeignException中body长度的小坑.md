---
title: FeignException中body长度的小坑
date: 2022-04-07 17:03:36
tags:
- feign
- spring cloud
categories:
- develop
---

前几天看到一个bug：别的模块通过feign调用我的模块报错，错误信息显示不完整··当然我是有点懵的：不应该啊，我看日志里边整个日志都是有的，怎么会用feign传过来就出了问题呢？然后第一反应就是肯定是feign限制了http传输过程中body的长度，当然这个第一反应也没有错。

我就开始在代码中定位问题，然而没想到我自己彻底走歪了：先到公共组件库里边发现公共组件对`Fallback`做了封装，写了一个专属的`fallbackFactory`，在这个里边把`errormsg`转成了`FeignException`的格式，在这个里边确实会对超长的body做压缩，变成`[response.status() during [response.request().httpMethod()] to [response.request().url()] [methodKey]: [body]]`的形式，如果body长度大于400，就会截取前200个`char`，然后在后边补上`...(xxx bytes)`。

我就想那我不转成`FeignException`不就完事了？一顿操作下去就搞定了，接下来一顿调试发现怎么改都不成，不对劲啊，怎么没有用呢··然后又想了想，发现我刚开始写的时候根本没看公共组件库，用的是原生的`fallbackFactory`。那就只有一个可能：feign会自己把`error`消息转换成`FeignException`类。

接下来我觉得这么搓肯定是因为我们用的版本太老了，新的版本里边这个长度肯定是可以配置的，就跑到github上一顿看··发现最新版本的确实有优化，然而只是把变量抽了出来，还是不可配置····

那没辙了，乖乖自己修改callback长度吧。