---
title: 如何写一个telegram input插件
date: 2021-02-19 23:02:34
tags:
- go
- telegram
categories:
- 指南
---

# telegraf introduce
telegraf [github](https://github.com/influxdata/telegraf) 地址
# why have this blog
我要写一个特殊监控，现有的telegraf插件又没有办法实现，所以就要自己写一个input plugin

先讲一下通常方法下，现有的input插件无法满足要求的情况下怎么做：

有一个[input.exec](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/exec)插件，可以获取命令执行后的结果，取到这个结果然后结合一下telegraf允许输入的格式，就可以了。无奈我写bash的水平实在是太低了，刚开始的时候也没有发现这个东西，所以就没有管这个，直接上手撸了个插件

<!--more-->

# how write a input plugin
先看一下[官方教程](https://github.com/influxdata/telegraf/blob/master/docs/INPUTS.md)吧

如果看懂的话大概就不太需要接下来的东西了

接下来是小白版的教程：

一个input plugin需要从以下做起：

1. 在 */telegraf/plugins/input/* 下增加插件文件夹
2. 编写插件go文件
3. 在 */telegraf/plugins/input/all/all.go*下增加编写好的路径
4. 重新编译telegraf
5. 运行

接下来主要看以下 插件文件的编写，以下称为 example.go

要先给定一个输入参数的结构体：

```go
type Info struct {
	Address  []string `toml:"address"`
}
```

这个结构体会作为很多函数的输入用：

描述：

```go
func (*Info) Description() string {
	return "return netconf result"
}
```

样例输入：

```go
var sampleConfig = `
Address = ["西街小学"]
`

func (*Info) SampleConfig() string {
	return sampleConfig
}
```

初始化：

```go
func init() {
	inputs.Add("sample", func() telegraf.Input {
		return &Info{}
	})
}
```

Gather函数：用来处理信息、将信息给后续组件进行处理

```go
func (a *Info) Gather(acc telegraf.Accumulator) error {
	//内容自由发挥，填写好addGauge的四个参数就ok
	fieldsG := map[string]interface{}{
		"address": a.Address,
	} 
	tags := map[string]string{}{}
	now := time.Now()
	acc.AddGauge("test", fieldsG, tags, now)
	return nil
}
```
将以上几个部分组合在一起就是一个最简单的input插件。最重要的就是gather函数，自己来组织fieldsG这个结构体，将要监控的信息填进去。



