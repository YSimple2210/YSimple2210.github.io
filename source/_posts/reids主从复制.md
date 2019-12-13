---
title: reids主从复制
date: 2019-12-12 22:21:12
tags: [主从复制]
categories: 
-Redis
---

# Redis主从复制介绍

## 什么是主从复制

<img src="redis主从复制\redis_copy.png"/>



## 为什么使用主从复制

> - Redis-server 单点故障
> - 单节点QPS有限

## 主从复制应用场景分析

> - 读写分离场景，规避Redis单机瓶颈
> - 故障切换，Master出问提后还有slave节点可以使用

# 搭建主从复制

主Redis Server 以普通模式启动，主要是启动从服务的方式

## 1.第一种方式：命令行

> 连接需要实现从节点的Redis,执行命令：slaveof [ip] [port]

## 2.第二种方式：redis.conf配置方式

> 配置文件中增加
>
> slaveof [ip] [port]
>
> 从服务器是否只读（默认yes）
>
> slave-read-only yes

## 退出主从集群的方式

> Slaveof no one

Redis命令: info 可查看Redis信息 

​		info Replication

新版主从复制 使用 ：

​		replicaof [ip] [port]

# 主从复制流程

> 1. 从服务器通过<font color="red">psync</font>命令发送服务器已有的同步进度（同步源ID/同步进度offset）
> 2. master收到请求，同步源为当前master，则更具偏移量增量同步
> 3. 同步源非当前master,则进入全量同步：master生成rdb,传输到slave,加载到slave内存

## 核心知识

>
>
>- Redis默认使用一步复制，slave和master之间一步地确认处理的数据量
>- 一个Master可以拥有多个Slave
>- slave可以接受其他slave,一个slave可以有下级sub slave
>- 主从同步过程在master侧是非阻塞的
>- slave初次同步需要删除旧数据，加载新数据，会阻塞到来的请求

<img src="redis主从复制\redisHexin.png"/>



# 主从复制应用场景

>-   主从复制可以用来支持读写分离
>-   slave设置为只读，可以用在数据安全的场景下
>-   可以使用主从来避免master持久化造成的开销。mater关闭持久化，slave配置为不定期保存或者启用AOF([可以看另一个文章](http://www.baidu.com))
>-   注意：重新启动master程序将从一个空数据集开始，如果一个slave试图与它同步，那么这个slave也会被清空

<img src="redis主从复制\redis2.png"/>

# 主从复制的注意事项

>   读写分离的场景
>
>   -   数据复制延迟导致读到的数据过期或者读不到数据（网络原因、slave阻塞）
>   -   从节点故障（多个client如何迁移？）

>   全量复制的情况下
>
>   -   第一次建立主从关系或者runid不匹配会导致全量复制
>   -   故障转移的时候也会出现全量复制

>   复制风暴
>
>   -   master故障重启后，如果slave节点较多。所有的slave都要复制，对服务器的性能，网络的压力都有很大影响。
>   -   一个服务器上部署多台master

>   写能力有限
>
>   -   主从复制还是只有一台master,写能力有限

>   master故障的情况下
>
>   -   如果master无持久化，slave开启持久化来保留数据的场景，不建议配置redis重启
>   -   如果redis自动重启，master启动后，无备份数据，可能导致集群数据丢失。

>   带有效期的key
>
>   -   slave不会让key过期，而是等待master 让key过期
>   -   在LUA脚本执行期间。不执行任何key过期操作。