---
title: MQ中间件通道状态STATUS(RETRYING)的问题分析与解决方法
date: 2017-09-19 10:58:16
category:
    - 架构&设计
    - 经典文摘
tags:
    - MQ
    - 消息中间件
---
这种问题一般发送在发送端，在我们发出启动通道的命令之后，通道进入binding的状态，若网络连接畅通并且通道定义正确，它进入正常running状态，如果出现了如下的一些问题，则通道进入retrying状态。
检查通道状态示例


```
$ runmqsc QMgrName
dis chs(C)
AMQ8417: Display Channel Status details.
   CHANNEL(C)                              XMITQ(QX)
   CONNAME(xxx.xxx.xxx.xxx (1416))         CURRENT
   CHLTYPE(SDR)                            STATUS(RETRYING)
```

 
原因可能有如下几种
1. 网络连接有问题
1. 通道定义不正确
1. 通道两端的消息序列号(Message Sequence Number)不匹配
1. 通道定义中的CONNAME(HostName (PortNumber))使用了主机名但是hosts文件中没有定义
1. 接收方不能连通
1. 接收端没有启动监听
1. 接收端端口占用（比如其它队列管理器占用了该端口）
 
解决方法
- 网络连接有问题
     检查通道定义包括网络不通，可使用  telnet  端口  测试连接
- 通道两端的消息序列号(Message Sequence Number)不匹配
     详细请参考本站文章：Websphere MQ消息序号Message Sequence详解
- 通道定义不正确
     检查通道配置，检查方法:
    $ runmqsc QMgrName
    dis chl(ChannelName)
- 通道定义中的CONNAME(HostName (PortNumber))使用了主机名但是hosts文件中没有定义
    检查通道定义，检查方法：
 

```
$ runmqsc QMgrName
    dis chl(ChannelName)
```

 
检查其中CONNAME是否使用了主机名，如果使用了，请检查/etc/hosts文件中是否有其定义。
- 接收方不能连通 和 F、接收端没有启动监听
检查方法：MQSC 中的测试通道命令PING，格式如下：

```
$ runmqsc QMgrName
     PING CHANNEL(channel_name) [DATALEN( 16 | integer)]
```

其中，DATALEN 表示 PING 数据包的大小，可以用 16 字节到 32,768 字节。
     PING 命令可以检查对方的队列管理器或端口监听器是否启动，也可以检查对方的通道定义是否正确。但不检查通道的通性状态。换句话说，PING CHANNEL 只检查通道能否连连通，而不检查目前是否连通。
- 接收端端口占用
     接收端相应的队列管理器停止监听，然后检查端口是否还在监听：
    
```
$ netstat -an|grep 端口号
```
