---
title: reids主从复制
date: 2019-12-12 22:21:12
tags: [主从复制]
categories: 
-Redis
---

# Redis主从复制介绍

## 什么是主从复制

<img src="reids主从复制\redis_copy.png"/>



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





