---
layout:     post
title:     「技术」Nginx的一些知识点
subtitle:   "从0开始"
date:       2017-04-19 21:00:00
author:     "Lucifa Diva"
header-img: "img/post-bg-apple-event-2015.jpg"
tags:
    - 技术
---


<div>
  <pre>
    Nginx进程模型：worker争抢互斥锁（1）accept_mutex（2）注册listenfd读事件—>accept连接—>读取请求—>解析请求—>处理请求—>产生数据—>返回给客户端—>断开连接
     每个worker一个主线程，异步非阻塞方式处理请求，select/poll/epoll/kqueue这样的系统调用，机制：同时监控多个事件，调用他们是阻塞的，但可以设置超时时间，在超时时间之内，如果有事件准备好了，就返回。可以理解为循环处理多个准备好的事件。并发数再多也不会导致无谓的资源浪费（上下文切换）。更多的并发数，只是会占用更多的内存而已。
     Web服务器事件：网络事件、信号、定时器。定时事件：维护定时器的红黑树

一、Nginx基本概念
     （1）connection：对TCP连接的封装，socket+读事件+写事件
          Nginx启动时master进程，初始化好监控的socket —> fork出多了子进程 —> 子进程accept成功  —> 得到建立好连接的socket —>创建ngx_connect_t结构体
          worker_connectons：每个进程支持最大连接数，每个worker有一个独立连接池，链表free_connections：空闲ngx_connection_t；反响代理服务器，每个并发会建立与客户端的连接和与后端服务器的连接，占用两个连接。
          多进程间连接平衡：当剩余idle连接数<总连接数/8,不参与accept竞争。

     注意，别增加无谓的上下文切换
     （2）request
     http请求：ngx_http_request_t对一个http请求的封装，请求行+请求头+请求体+响应行+响应头+响应体
     一个请求：ngx_http_init_request开始（设置事件为ngx_http_process_request_line）；ngx_http_read_request_header读取请求数据；ngx_http_parse_request_line解析请求数据，请求参数保存在ngx_http_request_t中
     读时间handler为ngx_http_process_request_headers..
     ngx_http_handler来真正开始处理一个完整的http请求,

     （3）请求相关keepalive
     （4）pipe
     （5）lingering_close

二、Nginx数据结构
     （1）ngx_str_t
     （2）ngx_pool_t:提供了一种机制，帮助管理一系列资源，src/core/ngx_palloc.h|c，
     （3）ngx_array_t：src/core/ngx_array.c|h
     （4）ngx_hash_t:解决冲突开链法，实际上是连续内存开数组，
     （5）ngx_hash_wildcard_t处理带有通配符的域名匹配问题
三、Nginx配置系统
     一个主配置文件+其他一些辅助配置文件
     每行配置，指令（指令包含空格一定要用引号）+指令参数（*个空格活TAB分开）格式
     指令上下文，多个作用域：（1）main:与业务无关，如工作进程数，运行身份（2）http:（3）server（4）location（5）mail
四、Nginx的模块化体系结构
     核心部分（实现底层通信协议，运行环境）+功能模块s

五、handler模块
     基本结构，hfilter进行处理。andler处理函数：直接生成内容；拒绝处理由后序handler处理；丢给filter进行后续处理。
     handler挂载：真正处理函数两种方式挂载到处理过程，一种按处理阶段挂在；按需挂载。

六、核心模块
     特别对于Linux，Nginx大部分event采用epoll EPOLLET（边沿触发）的方法来触发事件，只有listen端口的读事件是EPOLLLT（水平触发）。
     Nginx多进程的锁在底层默认是通过CPU自旋锁来实现。如果操作系统不支持自旋锁，就采用文件锁。
  </pre>
</div>


