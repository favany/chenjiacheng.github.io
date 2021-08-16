---
layout: post
categories: Golang
tags: [Go基础]
---

## Redis下载

官方网站：https://redis.io

![01.png](/static/images/20210816/01.png)

中文官方网站：http://www.redis.cn

![02.png](/static/images/20210816/02.png)

## 安装步骤

**1. 下载 `redis-6.0.6.tar.gz` 放 `/opt` 目录**

**2. 进入 `/opt` 目录下，执行解压命令：`tar -zxvf redis-6.0.6.tar.gz`**

**3. 解压完成后进入 `redis-6.0.6` 目录，执行 `make` 命令**

> 如果 make 安装失败，则需要安装或升级 gcc

**5. 如果 `make` 完成后继续执行 `make install`**

**6. 安装成功后查看 Redis 默认安装目录：`/usr/local/bin`**

```bash
[root@iZwz98zi7ua638a1dxozhyZ redis-6.0.6]# ls /usr/local/bin/
jemalloc-config   luajit        redis-benchmark  redis-sentinel
jemalloc.sh       luajit-2.0.4  redis-check-aof  redis-server
jeprof            mcrypt        redis-check-rdb
libmcrypt-config  mdecrypt      redis-cli
```

Redis 目录文件说明：

| 文件名          | 描述                               |
| --------------- | ---------------------------------- |
| redis-benchmark | 性能测试工具（服务启动起来后执行） |
| redis-check-aof | 修复有问题的 AOF 文件              |
| redis-check-rdb | 修复有问题的 dump.rdb 文件         |
| redis-sentinel  | Redis 集群使用                     |
| redis-server    | Redis 服务器启动命令               |
| redis-cli       | 客户端，操作入口                   |

## 前台启动

前台启动，命令行窗口不能关闭，否则服务器停止。

```bash
[root@iZwz98zi7ua638a1dxozhyZ ~]# redis-server
10223:C 16 Aug 2021 20:51:28.296 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
10223:C 16 Aug 2021 20:51:28.296 # Redis version=6.0.6, bits=64, commit=00000000, modified=0, pid=10223, just started
10223:C 16 Aug 2021 20:51:28.296 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 6.0.6 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 10223
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

10223:M 16 Aug 2021 20:51:28.297 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
10223:M 16 Aug 2021 20:51:28.297 # Server initialized
10223:M 16 Aug 2021 20:51:28.297 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
10223:M 16 Aug 2021 20:51:28.297 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
10223:M 16 Aug 2021 20:51:28.297 * Ready to accept connections
```

## 后台启动

**拷贝一份 `redis.conf` 到其它目录（备用）**

```bash
[root@iZwz98zi7ua638a1dxozhyZ ~]# cd /usr/local/bin/
[root@iZwz98zi7ua638a1dxozhyZ bin]# mkdir myredis
[root@iZwz98zi7ua638a1dxozhyZ bin]# cp /opt/redis-6.0.6/redis.conf myredis
[root@iZwz98zi7ua638a1dxozhyZ bin]# ls myredis/
redis.conf
```

**将 `redis.conf` 中的 `daemonize no` 改成 `yes`**

```bash
[root@iZwz98zi7ua638a1dxozhyZ bin]# vim myredis/redis.conf
```

**后台启动 Redis 服务**

```bash
[root@iZwz98zi7ua638a1dxozhyZ bin]# redis-server myredis/redis.conf
10276:C 16 Aug 2021 21:07:27.343 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
10276:C 16 Aug 2021 21:07:27.344 # Redis version=6.0.6, bits=64, commit=00000000, modified=0, pid=10276, just started
10276:C 16 Aug 2021 21:07:27.344 # Configuration loaded
```

**用客户端访问测试验证**

```bash
# 默认访问 6379 端口
[root@iZwz98zi7ua638a1dxozhyZ bin]# redis-cli
127.0.0.1:6379> ping
PONG

# 多个端口情况下，可指定端口
[root@iZwz98zi7ua638a1dxozhyZ bin]# redis-cli -p 6379
127.0.0.1:6379> ping
PONG
```

**关闭 Redis 服务**

```bash
# 单个实例关闭
[root@iZwz98zi7ua638a1dxozhyZ bin]# redis-cli shutdown

# 多个端口情况下，可指定端口关闭
[root@iZwz98zi7ua638a1dxozhyZ bin]# redis-cli -p 6379 shutdown

# 也可以进入终端后再关闭
[root@iZwz98zi7ua638a1dxozhyZ bin]# redis-cli
127.0.0.1:6379> shutdown
```

## 相关知识

Redis 默认16 个数据库，类似数组下标从 0 开始，初始默认使用 0 号库

**查看 `redis.conf` ，里面有默认的配置**

```bash
# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
databases 16
```

**使用命令 select <dbid> 来切换数据库。如：`select 8`**

```bash
127.0.0.1:6379> select 8
OK
```

**`dbsize` 查看当前数据库的 key 的数量**

```bash
127.0.0.1:6379[8]> dbsize
(integer) 0
```

**`flushdb` 清空当前库**

```bash
127.0.0.1:6379[8]> flushdb
OK
```

**`flushall` 清空全部的库**

```bash
127.0.0.1:6379[8]> flushall
OK
```

**Redis 是单线程 + 多路 IO 复用技术**

多路复用是指使用一个线程来检查多个文件描述符（Socket）的就绪状态，比如调用 select 和 poll 函数，传入多个文件描述符，如果有一个文件描述符就绪，则返回，否则阻塞直到超时。得到就绪状态后进行真正的操作可以在同一个线程里执行，也可以启动线程执行（比如使用线程池）。

**为什么 Redis 是单线程？**

我们首先要明白，Redis 很快！官方表示，因为 Redis 是基于内存的操作，CPU 不是 Redis 的瓶颈，Redis 的瓶颈最有可能是机器内存的大小或者网络带宽。既然单线程容易实现，而且 CPU 不会成为瓶颈，那就顺理成章地采用单线程的方案了！

Redis 采用的是基于内存的采用的是单进程单线程模型的 KV 数据库，由 C 语言编写，官方提供的数据是可以达到100000+ 的 QPS（每秒内查询次数）。这个数据不比采用单进程多线程的同样基于内存的 KV 数据库 Memcached 差！

**Redis 为什么这么快？**

Redis 核心就是如果数据全都在内存里，单线程的去操作就是效率最高的，为什么呢，因为多线程的本质就是 CPU 模拟出来多个线程的情况，这种模拟出来的情况就有一个代价，就是上下文的切换，对于一个内存的系统来说，它没有上下文的切换就是效率最高的。Redis 用单个 CPU 绑定一块内存的数据，然后针对这块内存的数据进行多次读写的时候，都是在一个CPU上完成的，所以它是单线程处理这个事。在内存的情况下，这个方案就是最佳方案。

因为一次 CPU 上下文的切换大概在 1500ns 左右。从内存中读取 1MB 的连续数据，耗时大约为 250us，假设1MB 的数据由多个线程读取了 1000 次，那么就有 1000 次时间上下文的切换，那么就有1500ns * 1000 = 1500us ，单线程的读完 1MB 数据才 250us，光时间上下文的切换就用了 1500us 了，还不算每次读一点数据 的时间。
