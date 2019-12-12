---
title: redis持久化机制
date: 2019-12-12 13:06:20
tags:[redis]
categories: 
-Redis
---



# Redis持久化介绍

Redis的数据都存放在内存中，如果没有配置持久化，redis重启后数据就会全丢失，开始持久化后，数据就会存放在磁盘中，当Redis重启后，就可以从磁盘中回复数据。

## Redis持久化方式

### RDB

>   RDB持久化方式能够在指定的时间间隔对你的数据进行快照存储

#### RDB方式

>   客户端直接通过命令<font color="red">BGSAVE</font>或者<font color="red">SAVE</font>来创建一个内存快照
>
>   -   bgsave调用fork来创建一个子进程，子进程负责将快照写入磁盘，而父进程而然继续处理命令。
>   -   SAVE执行SAVE命令过程中，不再响应其它命令

>   配置
>
>   在redis.conf中调整save配置选项,当在规定的时间内，Redis发生了写操作的个数满足条件会触发发生BESAVE命令
>
>   <img src="redis持久化机制\saveAndBgSave.png"/>

#### RDB优缺点

|           优点           |                             缺点                             |
| :----------------------: | :----------------------------------------------------------: |
|      对性能影响最小      |                        同步时数据丢失                        |
| RDB文件恢复比AOF要快的多 | 如果数据量大且CPU不够强，Redis在fork子进程可能会消耗很长的时间 |



### AOF

>   AOF持久化方式记录每次对服务器<font color="red">写</font>的操作,当服务器重启的时候会重新执行这些命令来恢复原始数据

#### AOF方式

>   开启AOF机制
>
>   1.  appendonly yes

>   AOF策略调整
>
>   1.  每次有数据修改发生时（appendfsync always）
>   2.  每秒钟同步一次,该策略为AOF的缺省策略（appendfsync everysec）
>   3.  从不同步，高效但是数据不会被持久化（appendfsync no）

#### AOF 优缺点

| 优点         | 缺点            |
| ------------ | --------------- |
| 最安全       | 文件体积大      |
| 容灾         | 性能消耗比RDB大 |
| 易读，可修改 | 恢复数据比RDB慢 |

# Redis内存分配



不同数据类型的大小限制

|         | 描述                                         |
| :------ | -------------------------------------------- |
| Lists   | 个数最大2^32-1个，也就是4294967295（40多亿） |
| Sets    | 个数最大2^32-1个，也就是4294967295（40多亿） |
| Hashes  | 个数最大2^32-1个，也就是4294967295（40多亿） |
| Strings | 一个value最大可以存储512M                    |

>   #最大内存控制
>
>   -   maxmemory 最大内存阈值
>   -   maxmemory-policy 到达阈值的执行策略

## 内存压缩

| 命令                         | 描述                    |
| ---------------------------- | ----------------------- |
| hash-max-zipmap-entries 512  | 配置字段最多512个       |
| hash-max-zipmap-value 64     | 配置value最大为64字节   |
| list-max-ziplist-entries 512 | 配置元素个数最多为512个 |
| list-max-ziplist-value 64    | 配置value最大为64字节   |
| set-max-intset-entries 512   | 配置元素个数最多为512个 |
| zset-max-ziplist-entries 128 | 配置元素个数最多为128个 |
| zset-max-ziplist-value 64    | 配置value最大为64字节   |

## 过期数据的处理策略（Redis内存满了会怎么样？）

### 1.主动处理

>   Redis主动触发检测Key是否过期，每秒执行10次。过程如下：
>
>   1.  从具有相关过期的密钥中测试20个随机密钥
>   2.  删除找到的所有密钥已过期
>   3.  如果超过25%的密钥已过期，从步骤1重新开始

### 2.被动处理

>   1.每次访问key的时候，发现超时后被动过期，清理掉

## 数据恢复阶段过期数据的处理策略

### RDB方式

>   1.  过期的Key不会被持久化到文件中
>   2.  载入时过期的key，会通过redis的主动和被动方式清理掉

### AOF方式

>   当Redis使用AOD方式持久化时，每次遇到过期的key,Redis会追加一条DEL命令到AOF文件
>
>   也就是：当我们顺序载入执行AOF命令文件就会删除过期的键

## Redis内存回收策略

>   配置文件中设置：maxmemory-policy noeviction
>
>   动态调整：config set maxmemory-policy noeviction

<img src="redis持久化机制\memory-policy.png"/>

### LRU算法

>   LRU(Least recentyl used ，最近最少使用)：根据数据的历史访问记录来进行淘汰数据
>
>   -   核心思想：如果数据最近被访问过，那么将来被访问的几率也更高
>   -   注意：Redis的LRU算法并非完整的实现,完整的LRU实现是因为需要太多的内存
>   -   方法：通过对少量keys进行取样（50%），然后挥手其中一个最好的key
>   -   配置方式：maxmemory-samples 5