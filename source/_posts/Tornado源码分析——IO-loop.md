---
title: Tornado源码分析——IO loop
date: 2017-01-26 13:29:19
tags: 
  - Python
  - Tornado
---

我们发现不管是哪种非阻塞I/O框架，都有一个主循环，不论是Node.js的event loop
还是Tornado/asyncio的IO [event] loop。

从手册知道，IO loop的实现在`tornado/ioloop.py`文件中，于是我们打开一探究竟。。

```python3
class IOLoop(Configurable):
    ...
```
我们很快找到了IOLoop类的定义，首先是一大堆常量表：

```python3
    ...
    # Constants from the epoll module
    _EPOLLIN = 0x001
    _EPOLLPRI = 0x002
    _EPOLLOUT = 0x004
    _EPOLLERR = 0x008
    _EPOLLHUP = 0x010
    _EPOLLRDHUP = 0x2000
    _EPOLLONESHOT = (1 << 30)
    _EPOLLET = (1 << 31)
    
    # Our events map exactly to the epoll events
    NONE = 0
    READ = _EPOLLIN
    WRITE = _EPOLLOUT
    ERROR = _EPOLLERR | _EPOLLHUP
    ...
```
