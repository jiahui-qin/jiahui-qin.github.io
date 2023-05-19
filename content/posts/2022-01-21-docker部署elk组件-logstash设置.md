---
title: docker部署elk组件&logstash设置
date: 2022-01-21 17:36:15
tags:
- elk
- docker
categories:
- develop
---

由于我们开发环境的elk坏了很久(直到我自己重新安装才知道是哪里坏了)，就计划再重新安装一遍elk，也学习一下elk的知识。

只是安装嘛··elk的基础知识就不介绍了，毕竟网上多的是，就说一下安装方案吧：

把Logstash,Elasticsearch,Kibana部署在一台机器上单独做日志服务，

filebeat 做为采集器部署分别部署在开发环境和测试环境两台服务器上，连接到logstash传递log

然后分两部分来看安装步骤：

<!--more-->

## filebeat 安装

随手整一个docker-compose：

```yml
version: "3"
services:
  filebeat:
    container_name: filebeat
    hostname: filebeat
    image: elastic/filebeat:7.16.3
    restart: always
    user: root
    volumes:
      - /data/logs:/data/logs
      - /data/elk/filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml
    network_mode: host
```
里边微微注意一下挂载目录就好了，也很直白嘛

然后就是`filebeat.yml`配置文件，给一个精简点的：
```yml
filebeat.config:
  modules:
    path: ${path.config}/modules.d/*.yml
    reload.enabled: false

processors:
  - add_cloud_metadata: ~
  - add_docker_metadata: ~

filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /data/logs/application/*/*.log
    exclude_lines: ['\sDEBUG\s\d']
    fields:
      docType: sys-log
    multiline:
      pattern: '^\[\S+:\S+:\d{2,}] '
      negate: true
      match: after

output.logstash:
  hosts: ["logstash:5044"]
  bulk_max_size: 2048
```
基本上也很简明易懂嘛，这里针对特定目录的日志给了一个标记，方便在logstash里边处理

## elk部署

elk部署最开始找了几个不靠谱教程，属实没意思，后来发现有一个[docker-elk](https://github.com/deviantony/docker-elk)的项目非常给力，直接clone下来就是一个部署的走起

具体怎么部署里边讲的也很清晰··我就稍微讲几个没有提到的东西

想要无密码登录，直接把所有配置里边和密码相关的都删了就完事，还要要删掉`elasticsearch.yml`里边的这几句：

```yml
xpack.license.self_generated.type: trial
xpack.security.enabled: true
xpack.monitoring.collection.enabled: true
```

还有一个花时间最多的地方就是logstash的配置上：

举个例子，我们一条日志是这样的：

`[log:111.11.11.11:0000] 2022-01-21 17:37:44.913 INFO 1 [] [XNIO-1 task-1] org.springframework.web.servlet.DispatcherServlet Completed initialization in 34 ms`

里边明显有一些比较重要的信息，比如模块名称，线程名，日志产生时间··这些要分析，然后不做处理的话都在message里边，肯定要logstash做一个处理，那就请出来`grok`来完成这个任务··

写这么一个匹配语句：

`\[%{NOTSPACE:appName}:%{NOTSPACE:serverIp}:%{NOTSPACE:serverPort}\] %{TIMESTAMP_ISO8601:logTime} %{LOGLEVEL:logLevel} %{WORD:pid} \[\] \[%{GREEDYDATA:threadName}\] %{NOTSPACE:classname} %{GREEDYDATA:message}`

就可以讲上边一个看起来很难懂的日志解析成这样简单的形式：

```
{
  "appName": "log",
  "pid": "1",
  "serverPort": "0000",
  "message": "Completed initialization in 34 ms",
  "threadName": "XNIO-1 task-1",
  "logTime": "2022-01-21 17:37:44.913",
  "logLevel": "INFO",
  "classname": "org.springframework.web.servlet.DispatcherServlet",
  "serverIp": "111.11.11.11"
}
```
那具体要怎么做呢：

在`docker-elk/logstash/pipeline/logstash.conf`里边加一个filter就好啦！

```conf
filter {
  if [fields][docType] == "sys-log" {
    grok {
      match => { "message" => "\[%{NOTSPACE:appName}:%{NOTSPACE:serverIp}:%{NOTSPACE:serverPort}\] %{TIMESTAMP_ISO8601:logTime} %{LOGLEVEL:logLevel} %{WORD:pid} \[\] \[%{GREEDYDATA:threadName}\] %{NOTSPACE:classname} %{GREEDYDATA:message}" }
      overwrite => ["message"]
    }
    date {
      match => ["logTime","yyyy-MM-dd HH:mm:ss.SSS"]
    }
    date {
      match => ["logTime","yyyy-MM-dd HH:mm:ss.SSS"]
      target => "timestamp"
      locale => "en"
      timezone => "+08:00"
    }
    mutate {  
      remove_field => "logTime"
      remove_field => "@version"
      remove_field => "host"
      remove_field => "offset"
    }
  }
}
```

这里用到了之前filebeat的`docType`标记来做处理，至于match里边的grok表达式，写好之后可以在kinbana里边的devtools验证。这里我有踩到一个坑：我有一个自己定义的`threadName`的正则表达式，写在了一个单独的pattern文件里，然而坑的是没有挂在的地方···在网上翻了好长时间也没有找到，如果有大佬知道，还请指教一下。最后还是用`GREADYDATA`来代替。

最后一个要注意的点就是索引模式：如果不对默认的output插件修改的话，那么es里边的所有item都是用`logstash`来开头的，就可以直接用`logstash*`这个匹配模式来做匹配，可以匹配到所有的item。但是如果修改了output，那么对应的索引模式也要修改。