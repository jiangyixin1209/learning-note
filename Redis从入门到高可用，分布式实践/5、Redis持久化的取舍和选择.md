# 持久化的作用

## 什么是持久化

redis将对数据的更新将异步的保存到磁盘上

## 主流持久化方式

* 快照
	* Mysql Dump
	* Redis RDB
* 写日志
	* MySQL Binlog
	* Hbase Hlog
	* Redis AOF

***

# Redis RDB

## RDB介绍

RDB其实是Redis创建一份当前的快照，在硬盘上创建一份RDB文件(二进制)
在Redis启动时候会载入RDB文件进行数据的恢复
此外，RDB文件还是Redis主从复制的媒介

## RDB的触发机制(主要的三种方式)

### save(同步)

``` shell
redis > save
```

触发save的方式就是客户端发送一条save命令即可，不过由于save是同步操作，因此会产生阻塞。  
在save大数据量时，Redis会先产生一个副本，当save结束时保存为RDB文件，如果先前存在则会覆盖。

### bgsave(异步)

``` shell
redis > bgsave
```

触发bgsave的方式是客户端发送一个bgsave命令。此时Redis会fork当前进程，然后使用fork的进程进行快照操作。

|命令|save|bgsave|
|:---|:---:|---:|
|IO类型|同步|异步|
|阻塞|是|是(阻塞发生在fork)|
|复杂度|O(n)|O(n)|
|优点|不会消耗额外内存|不会阻塞客户端命令|
|缺点|阻塞客户端命|需要fork,会消耗内存|

### 自动

在Redis的配置文件中进行设置

|配置|seconds|changes|
|:---|:---:|:---:|
|save|900|1|
|save|300|10|
|save|60|10000|


例如如果Redis在60s中修改了10000条数据，Redis会自动生成一份RDB文件。  
只要满足以上任何一个条件，Redis就会自动快照。save的保存方式是bgsave


## RDB触发机制(不容忽略方式)


* 全量复制：在Redis进行主从复制的时候，Redis会自动生成RDB文件

* debug reload：在使用debug级别的重启(不会清空内存中的数据)

* shutdown：在Redis关闭的时候会产生RDB文件

## Redis关于RDB的配置

### 基本配置

``` shell
save 900 1
save 300 10
save 60 10000

# rdb的文件名
dbfilename dump.rdp

# rdb文件存放的路径
dir ./

# 当bgsave发生错误时是否停止写入
stop-writes-on-bgsave-error yes

# 是否启用rdb文件压缩
rdbcompression yes

# 是否启用rdb文件检查
rdbchecksum yes

```

### 最佳配置

``` shell
# 一般会关闭自动快照
# save 900 1
# save 300 10
# save 60 10000

# rdb的文件名
# 采用Redis端口号来进行区分
dbfilename dump-${port}.rdp

# rdb文件存放的路径
# 将RDB放入其他目录的文件
dir /customPath

# 当bgsave发生错误时是否停止写入
stop-writes-on-bgsave-error yes

# 是否启用rdb文件压缩
rdbcompression yes

# 是否启用rdb文件检查
rdbchecksum yes

```

## RDB总结

* RDB是Redis内存到硬盘的快照，用于持久化。
* save通常会阻塞Redis。
* bgsave不会阻塞Redis，但是会fork新进程。
* save自动配置满足任一就会被执行(通常不会启用)。
* 有些触发机制不容忽视。

***

# Redis AOF

## RDB存在的问题

* 耗时、耗性能
	* O(n)数据：耗时。Redis需要将所有数据都存入RDB文件中。其中，Redis的写也是会消耗CPU。
	* fork(): 消耗内存。bgsave会产生Redis的一个子进程。虽然不会完全拷贝主进程和采用了copy-on-write策略，但是，如果数据量过大的情况下仍然会消耗大量内存。
	* Disl I/O: IO性能。
* 不可控，丢失数据

不可控，丢失数据可能会发生在以下场景

1、执行多个写命令  
2、满足RDB自动创建条件  
3、再次执行多个写命令  
4、服务器宕机

此时Redis就有可能丢失数据(使用bgsave也可能出现上述情况，因为无法预知何时宕机)

## 什么是AOF

AOF是Redis采用日志形式来进行持久化。  
原理：Redis将客户端每次发送的命令存储下来，等待恢复的时候读取AOF文件中的命令进行恢复
## AOF三种策略

Redis在将命令写入AOF文件中的时候，其实是现将命令写到缓冲区中，接着再根据不同的策略执行不同的操作

* always: 每条命令都是fsync写入到硬盘(保证数据不丢失)
* everysec: 每秒把缓冲区fsync到硬盘
* no: 由操作系统决定

|命令|always|everysec|no|
|:---|:---:|:---:|:---:|
|优点|不丢失数据|每秒一次fsync,丢失1秒的数据|不用管|
|缺点|IO开销较大, 一般sata盘只有几百TPS|丢失1秒的数据|不可控|


## AOF重写

AOF是将Redis的每条命令进行存储，这样随着命令的逐渐增加,AOF文件也会越来越大，这样在恢复的时候Redis可能出现g各种问题。为了解决此问题，Redis提供了AOF重写，即将过期的，无用的，重复的m命令进行压缩和优化。  

例如

``` shell
set key hello
set key hello1
set key hello2

# 以上三个命令会被压缩为
set key hello2

incr count
incr count

# 会被优化成
incr count 2

rpush list a
rpush list b
rpush list c

# 会被优化成
rpush list a b c
```

## AOF重写的作用

* 减少硬盘占用量
* 加快恢复速度

## AOF重写实现的两种方式

### bgrewriteaof

客户端通过发送bgrewriteaof命令来进行AOF重写

流程如下
* 客户端发送bgrewriteaof命令
* redis收到命令，返回OK响应，并且fork出子进程
* 子进程读取redis内存中数据，将其进行AOF重写后写入到aof文件

### AOF重写配置

|配置名|含义|
|:---|:---:|
|auto-aof-rewrite-min-size|AOF文件重写需要的尺寸|
|auto-aof-rewrite-percentage|AOF文件增长率|


|统计名|含义|
|:---|:---:|
|auto_current_size|AOF当前的尺寸(字节)|
|auto_base_size|AOF上次启动和重写的尺寸(字节)|

当同时满足以下情况时，会自动触发AOF重写

* aof_current_size > auto-aof-rewrite-min-size
* (aof_current_size - auto_base_size) / auto_base_size > auto-aof-rewrite-percentage

## AOF重写流程

![](http://p0lgavykh.bkt.clouddn.com/2018-01-27_121044.jpg)

## 配置

``` shell 
# 能够对AOF进行追加
appendonly yes
# AOF文件名
appendfilename "appendonly-${port}.aof"
# AOF的刷新策略
appendfsync everysec
# 存储路径
dir /customDir
# 在AOF重写时不要进行追加
no-appendfsync-on-rewrite yes

```

***

# RDB和AOF的选择


|命令|RDB|AOF|
|:---|:---:|:---:|
|启动优先级|低|高|
|体积|小|大|
|恢复速度|快|慢|
|数据安全性|丢失数据|由策略绝对|
|轻重|重|轻|