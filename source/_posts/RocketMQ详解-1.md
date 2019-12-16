---
title: RocketMQ详解(1)
date: 2019-12-16 12:12:44
tags: [消息中间件]
categories: -Rocket
---

# Rocket 入门

## RocketMQ 的特性

>   -   原生分布式
>   -   两种消息拉取
>   -   严格消息顺序
>   -   特有的分布式协调器
>   -   亿级消息堆积
>   -   组（Group）

## 基本概念

| 名词           | 概念                                                         |
| -------------- | ------------------------------------------------------------ |
| Producer       | 消息生产者，负责生产消息，一般由业务系统负责生产消息         |
| Consumer       | 消息消费者，负责消费消息，一般是后台系统负责异步消费         |
| Push Consumer  | 封装消息拉取，消费进程和内部                                 |
| Pull consumer  | 主动拉取消息，一旦拉取到消息，应用的消费进程进行初始化       |
| Producer Group | 一类Producer的集合名称，这类Producer通常发送一类消息，且发送逻辑一样 |
| Consumer Group | 一类Consumer的集合名称，这类Consumer通常消费一类消息，且消费逻辑一样 |
| Broker         | 消息中转角色，负责存储消息，转发消息，这里是RocketMQ Server  |
| Topic          | 消息的主题，用于定义并在服务端配置，消费者可以按照主题进行订阅（消息分类）通常一个系统一个主题 |
| Message        | 在生产者、消费者、中转站之间传递的消息，一个message必须属于一个Topic |
| namesrv        | 一个无状态的名称服务，可以集群部署，每个broker启动的时候都会向名称服务注册，主要是接收broker的注册，接收客户端的路由请求并返回路由信息 |
| offset         | 偏移量，消费者拉取消息时，需要知道上一次消费到了什么位置，这一次从哪里开始 |
| partition      | 分区，Topic物理上的分组，一个topic可以分成多个分区，每个分区是一个有序的队列。分区中的每条消息都会分配给一个有序的ID，也就偏移量 |
| Tag            | 用于对消息进行过滤，理解为message的标记，同一业务不同目的的message可以用相同的topic,但是可以用tag来分区 |
| key            | 消息的key 字段是为了唯一表示消息的。方便查问题，不是说必须设置，只是说设置为了方便开发和运维定位问题。比如：这个KEY可以是订单ID等 |

## 安装RocketMQ

>   1.  解压 
>
>       unzip -d /usr rocketmq.zip

>   启动nameServer 
>
>   nohup sh bin/nameserver > ~/logs/rocketmqlogs/nameserver.log 2>&1 &

>   启动broker
>
>   nohup sh bin/mqbroker -n localhost:9876 > ~/logs/rocketmqlogs/broker.log 2>&1 &

在broker.conf 中配置brokerIP1 =xxx.xxx.xx.xxx 可以让外网访问





