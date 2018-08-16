---
title: Redis数据持久化的方式
comments: true
date: 2018-08-16 18:17:33
toc: true
tags:
- 持久化
categories: 
- Rdis
---
Redis 提供了三种数据持久化的方式将数据存储在磁盘中，一种叫快照（RDB），另一种叫只追加文件（AOF），还有一种是RDB-AOF混合持久化。
### 快照RDB
Redis通过创建快照的方式获取某一时刻Redis中所有数据的副本。
#### 手动设置Redis快照名及路径：
redis.conf
```
# RDB文件名 
dbfilename "dump.rdb" 
# RDB文件和AOF文件路径 
dir "/usr/local/var/db/redis"
```
#### Redis快照RDB创建
1. 客户端直接通过命令BGSAVE(background save 推荐)或者SAVE来创建一个快照。BGSAVE不影响主进程的工作。
2. 在redis.conf中设置save配置选项（应用开发中比较常用）
```
# 当在规定的时间内，Redis发生了写操作的个数满足条件，会触发发生BGSAVE命令。 
# save <seconds> <changes> 
# 当用户设置了多个save的选项配置，只要其中任一条满足，Redis都会触发一次BGSAVE操作，比如：900秒之内至少一次写操作、300秒之内至少发生10次写操作、60秒之内发生至少10000次写操作都会触发发生快照操作 
save 900 1 
save 300 10 
save 60 10000
```
3. 当Redis通过shutdown命令关闭服务器请求时，会执行SAVE命令创建一个快照，如果使用kill -9 PID将不会创建快照
4. 主从同步时
### 只追加文件（AOF）
在执行写命令时，AOF(appendonly)持久化会将执行的写命令也写到AOF文件的末尾，以此来记录数据的变化。换句话说，将AOF文件中包含的内容重新执行一遍，就可以回复AOF文件所记录的数据集。
```
# redis默认关闭AOF机制，可以将no改成yes实现AOF持久化 
appendonly no 
# AOF文件 
appendfilename "appendonly.aof" 
# AOF持久化同步频率，always表示每个Redis写命令都要同步fsync写入到磁盘中，但是这种方式会严重降低redis的速度；everysec表示每秒执行一次同步fsync，显示的将多个写命令同步到磁盘中；no表示让操作系统来决定应该何时进行同步fsync，Linux系统往往可能30秒才会执行一次
# appendfsync always 
appendfsync everysec 
# appendfsync no 
# 在日志进行BGREWRITEAOF时，如果设置为yes表示新写操作不进行同步fsync，只是暂存在缓冲区里，避免造成磁盘IO操作冲突，等重写完成后在写入。redis中默认为no 
no-appendfsync-on-rewrite no 
# 当前AOF文件大小是上次日志重写时的AOF文件大小两倍时，发生BGREWRITEAOF操作。 auto-aof-rewrite-percentage 100#当前AOF文件执行BGREWRITEAOF命令的最小值，避免刚开始启动Reids时由于文件尺寸较小导致频繁的BGREWRITEAOF。 
auto-aof-rewrite-min-size 64mb 
# Redis再恢复时，忽略最后一条可能存在问题的指令(因为最后一条指令可能存在问题，比如写一半时突然断电了) aof-load-truncated yes 
```
### RDB-AOF混合持久化
#Redis4.0新增RDB-AOF混合持久化格式，在开启了这个功能之后，AOF重写产生的文件将同时包含RDB格式的内容和AOF格式的内容，其中RDB格式的内容用于记录已有的数据，而AOF格式的内存则用于记录最近发生了变化的数据，这样Redis就可以同时兼有RDB持久化和AOF持久化的优点（既能够快速地生成重写文件，也能够在出现问题时，快速地载入数据）。
aof-use-rdb-preamble no

