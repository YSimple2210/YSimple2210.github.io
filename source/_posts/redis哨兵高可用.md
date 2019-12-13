---
title: redis哨兵高可用
date: 2019-12-13 12:53:23
tags: [redis哨兵]
categories: -Redis
---

# Redis哨兵

如果选择master / 哨兵的集群

## 哨兵（Sentinel）机制核心作用

<img src="redis哨兵高可用\1.png">

## 服务发现和健康检查流程

<img src="redis哨兵高可用\health.png">

## 故障切换流程

<img src="redis哨兵高可用\qiehuan.png">

## 7大核心概念

>1.  哨兵如何知道Redis主从信息（自动发现机制 [info replication]）
>2.  什么是master主观下线
>3.  什么是客观下线
>4.  哨兵之间如何通信（哨兵之间的自动发现）
>5.  哪个哨兵负责故障转移（哨兵领导选举机制）
>6.  slave选举机制
>7.  最终主从切换过程

#### 1.哨兵如何知道Redis主从信息

>   哨兵配置文件中保存着主从集群中master的信息，可以通过info命令，进行主从信息自动发现

#### 2.什么是主观下线

>   主观下线：单个哨兵自身认为redis实例已经不能提供服务
>
>   检测机制：哨兵向redis发送ping命令， + PONG、-LOADNG、-MASTERDOWN这三种情况下认为是正常，其它回复均为无效。
>
>   对应的配置文件中内容：sentinel down-after-milliseconds mymaster 1000

#### 3.客观下线

<img src="redis哨兵高可用\客观下线.png">

>   客观下线：一定数量值的哨兵认为master已经下线
>
>   检测机制：当哨兵主观认为master下线后，则会通过SENTINEL is-master-down-by-addr 命令，询问其它哨兵是否认为master下线。如果达成共识（达到quorum个数），就会认为master节点客观下线。开始故障转移
>
>   对应配置文件内容：sentinel monitor mymaster [ip] [quorum]

#### 4.哨兵之间如果通信

<img src="redis哨兵高可用\哨兵通信.png">

>   1.哨兵之间的自动发现（注册订阅）

#### 5.哨兵之间的选举机制

>基于Raft算法实现的选举机制：
>
>1.  拉票阶段：每个哨兵节点希望自己成为领导者
>2.  sentinel节点收到拉票命令后，如果没有收到或同意其它sentinel节点的请求，就同意该sentinel节点的请求（每个sentinel只持有一个同意票数）
>3.  如果sentinel节点发现自己的票数已经超过一半的数值，那么它就成为领导者，去执行故障转移
>4.  投票结束后、如果超过failover-timeout的时间内，没有进行实际的故障转移，则重新拉票选举。

#### 6.slave选举方案

按顺序依次筛选

>   slave节点状态，非S_DOWN,O_DOWN,DISCONNECTED
>
>   判断规则：（down-after-milliseconds * 10） + milliseconds_snice_master_is_in_SDOWN_state
>
>   SENTINEL slave mymaster

>   优先级
>   redis.conf中的一个配置：slave-priority 值越小，优先级越高

>   数据同步情况
>
>   Replication offset processed

>   最小的run id
>
>   run id比较方案：字典顺序 ,ASCII码

#### 7.最终主从切换过程

>   针对即将成为master的slave节点。将其撤出主从集群
>
>   自动执行：slaveof no one 

>    针对其它slave节点，使他们成为新的master的从属
>
>   自动执行：slaveof new_master_host new master_port

