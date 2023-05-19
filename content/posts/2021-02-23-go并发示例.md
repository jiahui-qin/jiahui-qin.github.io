---
title: go并发示例
date: 2021-02-23 15:31:09
tags:
- go
- 并发
categories:
- go初学者教程
---

在这片文章主要记录几个并发编程的示例，在写一些理解

[go语言中常见的并发模式](https://zhuanlan.zhihu.com/p/74655793)

[go 生产者消费者模型](https://www.cnblogs.com/fengchuiyizh/p/12299630.html)

我现阶段需要理解的核心还是通道的使用：

通道是有方向的，用来在主进程与子进程之间传递消息，接下来主要还是要理解以下并发编程到底是怎么做的。


<!--more-->



````go
import (
	"fmt"
	"sync"
)

func main() {
	var wg sync.WaitGroup
	// 开N个后台打印线程
	var res []int
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func() {
			res = append(res, i)
			wg.Done()
		}()
	}
	// 等待N个后台线程完成
	wg.Wait()
	fmt.Println(res)
}

input：
[10 10 10 10 10 10 10 10 10 10]
````

在这里最后的输出全都是10，在循环中把所有的线程都启动了起来再执行。


-----

生产者-消费者模式

````go
func Producer(ch chan int) {
	//生产者，把i传到ch里边
	for i := 1; i <= 10; i++ {
		ch <- i
	}
	close(ch)
}

func Consumer(id int, ch chan int, done chan bool) {
	//消费者，接受ch，并进行消费，done是完成的标志
	for {
		value, ok := <-ch
		if ok {
			fmt.Printf("id: %d, recv: %d\n", id, value)
		} else {
			fmt.Printf("id: %d, closed\n", id)
			break
		}
	}
	done <- true
}

func main() {
	ch := make(chan int, 3)

	coNum := 2
	done := make(chan bool, coNum)
	for i := 1; i <= coNum; i++ {
		go Consumer(i, ch, done)
	}

	go Producer(ch)

	//这里要拿到done的结果，在这里要求完成？
	for i := 1; i <= coNum; i++ {
		fmt.Println(<-done)
	}
}
`````


-----
并发版本的go素数筛

````go
func GenerateNatural() chan int {
	ch := make(chan int)
	go func() {
		for i := 2; ; i++ {
			fmt.Println(i)
			ch <- i
		}
	}()
	return ch
}

func PrimeFilter(in <-chan int, prime int) chan int {
	out := make(chan int)
	go func() {
		for {
			if i := <-in; i%prime != 0 {
				out <- i
			}
		}
	}()
	return out
}

func main() {
	ch := GenerateNatural() // 自然数序列: 2, 3, 4, ...
	for i := 0; i < 100; i++ {
		prime := <-ch // 新出现的素数
		fmt.Printf("%v: %v\n", i+1, prime)
		ch = PrimeFilter(ch, prime) // 基于新素数构造的过滤器
	}
}

````