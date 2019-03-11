---
title: ZooKeeper-分布式应用程序协调服务
date: 2019-03-06 14:51:32
tags: 
    - ZooKeeper
    - 开源软件
categories: ZooKeeper
---

[参考链接](http://developer.51cto.com/art/201809/583184.htm)

## 基本概念
Apache ZooKeeper is an effort to develop and maintain an open-source server which enables highly reliable distributed coordination.

Apache ZooKeeper致力于开发和维护一个支持高度可靠的分布式协调的开源服务器。
<!-- more --> 
Zookeeper是一个典型的分布式数据一致性解决方案，分布式应用程序可以基于Zookeeper实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master选举、分布式锁和分布式队列等功能。  


## Znode

在ZooKeeper中，“节点”分为两种类型：
- 第一类是指构成集群的机器，称之为机器节点；
- 第二类则是指数据模型中的数据单元，我们称之为数据节点-Znode。

ZooKeeper将所有的数据存储在内存中，数据模型是一棵树（Znode Tree）

## ZooKeeper集群介绍介绍

ZooKeeper集群中的所有机器通过Leader选举过程来选定一台称为“Leader”的机器，Leader既可以为客户端提供写服务又能提供读服务。出Leader外，Follower和Observer只能提供读服务。

Follower与Observer的唯一区别在于Observer机器不参与Leader的选举过程，也不参与写操作的“过半写成功”策略，因此Observer机器可以在不影响写性能的情况下提升集群的读性能。

## ZAB协议

ZAB（ZooKeeper Atomic Broadcast，ZooKeeper原子广播）协议是为分布式协调服务ZooKeeper专门设计的一种支持崩溃恢复的原子广播协议。

在ZooKeeper中，主要依赖ZAB协议来实现分布式数据一致性，基于该协议ZooKeeper实现了一种主备模式的系统架构来保持集群中各个副本之间的数据一致性。

ZAB协议包括两种基本模式，分别是崩溃恢复和消息广播。  
当整个服务框架在启动过程中，或是当Leader服务器出现网络中断、崩溃退出与重启等异常情况是，ZAB协议就会进入恢复模式并选举出新的Leader服务器。  
当选举产生的新的Leader服务器，同时集群中已经有过半的机器与新Leader服务器完成状态同步之后，ZAB协议就会退出恢复模式。其中，所谓的状态同步是指数据同步，也就是保证集群中过半的机器与该Leader服务器的数据状态一致。  
当集群中已经有过半的Follower服务器完成与Leader服务器的状态同步，那么整个服务框架就可以进入消息广播模式了。  
当一台同样遵守ZAB协议的服务器启动后加入到集群中时，如果此时集群中已经存在一个Leader服务器在负责消息广播，那么新加入的服务器就会自觉地进去数据恢复模式：找到Leader所在的服务器，并与其进行数据同步，然后一起参与到消息广播流程中去。

正如上文所述，ZooKeeper被设计成只允许唯一的Leader服务器进行事务请求的处理，Leader服务器在接收到客户端的事务请求后，会生成对应的事务提案并发起一轮广播协议。如果集群中的其他机器接收到客户端事务请求，那么这些非Leader服务器会首先将这个事务转发给Leader服务器。

## 应用

在我们的智慧教育示范应用中，主要是使用Kafka收集用户日志，而ZooKeeper就担任了服务生产者和消费者的注册中心。  
服务生产者将自己提供的服务注册到ZooKeeper中心，服务消费者在进行服务调用的时候先到ZooKeeper中查找服务，获取到服务生产者的详细信息之后，再去调用服务生产者的内容和数据。

