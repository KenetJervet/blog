---
title: 可伸缩的事件多路复用：epoll vs. kqueue
date: 2018-01-28 19:47:09
tags:
  - Linux
  - BSD
  - Asynchronous
---

> 本文是[http://people.eecs.berkeley.edu/~sangjin/2012/12/21/epoll-vs-kqueue.html](http://people.eecs.berkeley.edu/~sangjin/2012/12/21/epoll-vs-kqueue.html)的中文翻译。
> 虽然我很希望能有自己的想法和见解，然而无奈，既不知其然也不知其所以然。必须正视自己与别人的差距。

基本上，我对Linux的好感要大于BSD，然而BSD的kqueue却是我非常希望能够移植到Linux上的。

## 什么是事件多路复用？

比如说你有一个简单的Web服务器，有两个打开的连接（套接字）。当从任一连接接收到HTTP请求时，服务器应该给客户端发出响应。然而你不知道这两个连接中谁先发起请求，以及在什么时间发起请求。由于BSD套接字的调用是阻塞的，也就意味着当你在其中一个连接上调用`recv()`时，就不能再响应另一个连接发出的请求。此时，你需要祭出I/O多路复用。

一个简单暴力的I/O多路复用的实现方式是为每个连接创建一个进程/线程。这样一个连接的阻塞就不会影响到其他连接。这样做其实是相当于把处理调度/复用的繁杂任务抛给操作系统内核。多线程架构成本较高，内核维护数量较多的线程需要大量的工作。为每个线程创建额外的调用栈加大了内存开销，同时也降低了CPU缓存的访问局部性。

那么问题来了，如何能在不为每连接创建一个线程的基础上实现多路复用呢？简单一点，你可以使用“忙碌等待”，在每个连接上调用非阻塞接口并进行轮询，然而这极大地浪费了计算资源。我们需要知道的是哪个连接在什么时刻准备好数据。因此内核提供了一个应用和内核之间的独立通道，当你所关心的任意套接字准备就绪时，就会通知应用程序。这就是`select()`/`poll()`的工作原理，基于“准备就绪”模型。

## 回顾select()

`select()`和`poll()`的工作方式非常类似。先看一下`select()`函数长什么样子。

```c
select(int nfds, fd_set *r, fd_set *w, fd_set *e, struct timeval *timeout)
```
使用`select()`需要提供三个“关注集”：`r`，`w`和`e`（文件描述符的位图表示格式）。比如，如果你想从6号描述符上读取文件，就在`r`的第6位上置1。这个调用是阻塞的，直到关注集中的一个或多个描述符准备就绪，因此你可以在无阻塞的情况下对这些描述符进行操作。

当谈到可伸缩性时，我们发现这个模型有四个弊端：

1. 这些位图是固定大小的（FD_SETSIZE，通常是1024）。虽然存在绕过这一限制的方式。
1. 由于内核会覆盖这些位图参数，因此每次调用都需要重新填写关注集。
1. 对于每次调用，应用程序和内核都需要扫描整个位图来得知哪些描述符属于关注集，哪些是结果集。对于结果集来说尤其低效，因为通常结果集是稀疏的（每一时刻只有少数几个描述符是就绪的）。
1. 内核会遍历整个关注集来得知哪些描述符就绪。每次调用都会遍历。若没有任何描述符就绪，内核会重新遍历，并为每个套接字注册一个内部事件处理函数。

## 回顾poll()

`poll()`的设计解决了上面的一部分问题：
```c
poll(struct pollfd *fds, int nfds, int timeout)

struct pollfd {
    int fd;
    short events;
    short revents;
}
```
`poll()`不使用位图，而是文件描述符数组（解决了上面第一个问题）。将关注集（events）和结果集（revents）分离，也可以解决第二个问题，只要应用程序正确地维护和重用数组。The issue #3 could have been fixed if poll separated the array, not the field. The last issue is inherent and unavoidable, as both select() and poll() are stateless; the kernel does not internally maintain the interest sets.
