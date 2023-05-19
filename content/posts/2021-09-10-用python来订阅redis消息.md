---
title: 用python来订阅查看redis消息
date: 2021-09-10 18:59:34
tags:
- python
categories:
- python小工具
---

这其实是个以前写的小工具，不过写都写了，就想着完善一下再放出来。以前写的只是简陋的在命令行里边呈现，现在计划搞上html页面，可以方便查看！应该也不用数据库把，只看打开这一次的消息不就行了。先把以前搞的代码贴上来吧，后续有时间再做完善：

这个的py版本是3.x

main.py
```python
#!/usr/bin/env python
# -*- coding:utf8 -*-

from RedisHelper import RedisHelper

obj = RedisHelper()
redis_sub = obj.subscribe()

while True:
    msg = eval(str(redis_sub.parse_response()))
    print(msg[1])
    print(msg[2])
    print()   #[b'message', b'fm104.5', b'who are you?']
    # print(msg[2].decode('utf8'))
```

redisHelper.py
```python
#!/usr/bin/env python
# -*- coding:utf8 -*-

import redis

class RedisHelper(object):

    def __init__(self):
        self.__conn = redis.Redis(host='127.0.0.1',port = '3379')   #连接本机，ip不用写

    def subscribe(self):
        pub = self.__conn.pubsub()
        pub.subscribe("UP")
        pub.subscribe("DOWN")
        pub.subscribe("START")
        pub.subscribe("CLOSE")
        pub.parse_response()  #准备好监听(再调用一次就是开始监听)
        return pub
```
