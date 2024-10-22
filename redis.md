# Redis配置

## network

bind * -::1

`bind * -::1` 是 Redis 配置文件中的一项设置，主要用于控制 Redis 服务器监听的网络接口地址。具体解释如下：

- `bind *`: 这一部分告诉 Redis 监听所有可用的网络接口。这意味着 Redis 可以接受来自任何 IP 地址的连接，不仅限于本地或特定的网络接口。通常，使用 `*` 来监听所有网络接口是为了让 Redis 可以被远程访问。
  
- `-::1`: 这一部分是为了禁用对 IPv6 本地回环地址 (`::1`) 的监听。IPv6 中，`::1` 是本地回环地址，类似于 IPv4 中的 `127.0.0.1`。如果 Redis 绑定到 `::1`，它只能接受来自本地 IPv6 地址的连接。通过使用 `-::1`，你可以显式禁用对这个地址的监听。

```yml
###################################  NETWORK ###################################
 
#指定 redis 只接收来自于该IP地址的请求，如果不进行设置，那么将处理所有请求
bind 127.0.0.1
 
#是否开启保护模式，默认开启。要是配置里没有指定bind和密码。#开启该参数后，redis只会本地进行访问，拒绝外部访问。要是开启了密码和bind，可以开启。否则最好关闭，设置为no
protected-mode yes
 
#redis监听的端口号
port 6379
 
#此参数确定了TCP连接中已完成队列(完成三次握手之后)的长度， 当然此值必须不大于Linux系统定义的/proc/sys/net/core/somaxconn值，默认是511，而Linux的默认参数值是128。#当系统并发量大并且客户端速度缓慢的时候，可以将这二个参数一起参考设定。该内核参数默认值一般是128，对于负载很大的服务程序来说大大的不够。一般会将它修改为2048或者更大。#在/etc/sysctl.conf中添加:net.core.somaxconn = 2048，然后在终端中执行sysctl -p
tcp-backlog 511
 
#此参数为设置客户端空闲超过timeout，服务端会断开连接，为0则服务端不会主动断开连接，不能小于0
timeout 0
 
#tcp keepalive参数。如果设置不为0，就使用配置tcp的SO_KEEPALIVE值，使用keepalive有两个好处:检测挂掉的对端。降低中间设备出问题而导致网络看似连接却已经与对端端口的问题。在Linux内核中，设置了keepalive，redis会定时给对端发送ack。#检测到对端关闭需要两倍的设置值
tcp-keepalive 300
 
#是否在后台执行，yes：后台运行；no：不是后台运行
daemonize yes
 
#redis的进程文件
pidfile /var/run/redis/redis.pid
 
#指定了服务端日志的级别。级别包括：debug（很多信息，方便开发、测试），verbose（许多有用的信息，但是没有debug级别信息多），notice（适当的日志级别，适合生产环境），warn（只有非常重要的信息）
loglevel notice
 
#指定了记录日志的文件。空字符串的话，日志会打印到标准输出设备。后台运行的redis标准输出是/dev/null
logfile /usr/local/redis/var/redis.log
 
 
#是否打开记录syslog功能
#syslog-enabled no
 
#syslog的标识符。
#syslog-ident redis
 
#日志的来源、设备
#syslog-facility local0
 
#数据库的数量，默认使用的数据库是0。可以通过”SELECT 【数据库序号】“命令选择一个数据库，序号从0开始
databases 16
```

## SNAPSHOTTING

```yml
###################################  SNAPSHOTTING  ###################################
 
#RDB核心规则配置 save <指定时间间隔> <执行指定次数更新操作>，满足条件就将内存中的数据同步到硬盘中。#官方出厂配置默认是 900秒内有1个更改，300秒内有10个更改以及60秒内有10000个更改，则将内存中的数据快照写入磁盘。
#若不想用RDB方案，可以把 save "" 的注释打开，下面三个注释
save 900 1
save 300 10
save 60 10000
 
#当RDB持久化出现错误后，是否依然进行继续进行工作，yes：不能进行工作，no：可以继续进行工作，可以通过info中的rdb_last_bgsave_status了解RDB持久化是否有错误
stop-writes-on-bgsave-error yes
 
#配置存储至本地数据库时是否压缩数据，默认为yes。Redis采用LZF压缩方式，但占用了一点CPU的时间。#若关闭该选项，但会导致数据库文件变的巨大。建议开启。
rdbcompression yes
 
#是否校验rdb文件;从rdb格式的第五个版本开始，在rdb文件的末尾会带上CRC64的校验和。#这跟有利于文件的容错性，但是在保存rdb文件的时候，会有大概10%的性能损耗，所以如果你追求高性能，可以关闭该配置
rdbchecksum yes
 
#指定本地数据库文件名，一般采用默认的 dump.rdb
dbfilename dump.rdb
 
#数据目录，数据库的写入会在这个目录。rdb、aof文件也会写在这个目录
dir /usr/local/redis/var

```

## REPLICATION

```yml
################################# REPLICATION #################################
 
# 复制选项，slave复制对应的master。
# replicaof <masterip> <masterport>
 
#如果master设置了requirepass，那么slave要连上master，需要有master的密码才行。
#masterauth就是用来配置master的密码，这样可以在连上master后进行认证。
# masterauth <master-password>
 
#当从库同主机失去连接或者复制正在进行，从机库有两种运行方式：
#1) 如果slave-serve-stale-data设置为yes(默认设置)，从库会继续响应客户端的请求。
#2) 如果slave-serve-stale-data设置为no，INFO,replicaOF, AUTH, PING, SHUTDOWN, REPLCONF, ROLE, CONFIG,SUBSCRIBE, UNSUBSCRIBE,
#PSUBSCRIBE, PUNSUBSCRIBE, PUBLISH, PUBSUB,COMMAND, POST, HOST: and LATENCY命令之外的任何请求都会返回一个错误”SYNC with master in progress”。
replica-serve-stale-data yes
 
#作为从服务器，默认情况下是只读的（yes），可以修改成NO，用于写（不建议）
#replica-read-only yes
 
# 是否使用socket方式复制数据。目前redis复制提供两种方式，disk和socket。
#如果新的slave连上来或者重连的slave无法部分同步，就会执行全量同步，master会生成rdb文件。
#有2种方式：
#disk方式是master创建一个新的进程把rdb文件保存到磁盘，再把磁盘上的rdb文件传递给slave。disk方式的时候，当一个rdb保存的过程中，多个slave都能共享这个rdb文件。
#socket是master创建一个新的进程，直接把rdb文件以socket的方式发给slave。socket的方式就的一个个slave顺序复制。
#在磁盘速度缓慢，网速快的情况下推荐用socket方式。
repl-diskless-sync no 
#diskless复制的延迟时间，防止设置为0。一旦复制开始，节点不会再接收新slave的复制请求直到下一个rdb传输。
所以最好等待一段时间，等更多的slave连上来
repl-diskless-sync-delay 5
 
#slave根据指定的时间间隔向服务器发送ping请求。时间间隔可以通过 repl_ping_slave_period 来设置，默认10秒。
# repl-ping-slave-period 10
 
# 复制连接超时时间。master和slave都有超时时间的设置。#master检测到slave上次发送的时间超过repl-timeout，即认为slave离线，清除该slave信息。#slave检测到上次和master交互的时间超过repl-timeout，则认为master离线。#需要注意的是repl-timeout需要设置一个比repl-ping-slave-period更大的值，不然会经常检测到超时
# repl-timeout 60
 
#是否禁止复制tcp链接的tcp nodelay参数，可传递yes或者no。默认是no，即使用tcp nodelay。#如果master设置了yes来禁止tcp nodelay设置，在把数据复制给slave的时候，会减少包的数量和更小的网络带宽。#但是这也可能带来数据的延迟。默认我们推荐更小的延迟，但是在数据量传输很大的场景下，建议选择yes
repl-disable-tcp-nodelay no
 
#复制缓冲区大小，这是一个环形复制缓冲区，用来保存最新复制的命令。#这样在slave离线的时候，不需要完全复制master的数据，如果可以执行部分同步，只需要把缓冲区的部分数据复制给slave，就能恢复正常复制状态。#缓冲区的大小越大，slave离线的时间可以更长，复制缓冲区只有在有slave连接的时候才分配内存。#没有slave的一段时间，内存会被释放出来，默认1m
# repl-backlog-size 1mb
 
# master没有slave一段时间会释放复制缓冲区的内存，repl-backlog-ttl用来设置该时间长度。单位为秒。
# repl-backlog-ttl 3600
 
# 当master不可用，Sentinel会根据slave的优先级选举一个master。#最低的优先级的slave，当选master。而配置成0，永远不会被选举
replica-priority 100
 
#redis提供了可以让master停止写入的方式，如果配置了min-replicas-to-write，健康的slave的个数小于N，mater就禁止写入。#master最少得有多少个健康的slave存活才能执行写命令。#这个配置虽然不能保证N个slave都一定能接收到master的写操作，但是能避免没有足够健康的slave的时候，master不能写入来避免数据丢失。#设置为0是关闭该功能
# min-replicas-to-write 3
 
# 延迟小于min-replicas-max-lag秒的slave才认为是健康的slave
# min-replicas-max-lag 10
 
# min-replicas-max-lag 设置1或设置0禁用这个特性。
# Setting one or the other to 0 disables the feature.
# By default min-replicas-to-write is set to 0 (feature disabled) and
# min-replicas-max-lag is set to 10.
```

## security

```yml
#requirepass配置可以让用户使用AUTH命令来认证密码，才能使用其他命令。#这让redis可以使用在不受信任的网络中。为了保持向后的兼容性，可以注释该命令，因为大部分用户也不需要认证。#使用requirepass的时候需要注意，因为redis太快了，每秒可以认证15w次密码，简单的密码很容易被攻破，所以最好使用一个更复杂的密码
requirepass foobared
#把危险的命令给修改成其他名称。比如CONFIG命令可以重命名为一个很难被猜到的命令，这样用户不能使用，而内部工具还能接着使用
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
#设置成一个空的值，可以禁止一个命令
# rename-command CONFIG ""
```

## clients

```yml
# 设置能连上redis的最大客户端连接数量。默认是10000个客户端连接。
# 由于redis不区分连接是客户端连接还是内部打开文件或者和slave连接等，所以maxclients最小建议设置到32。
# 如果超过了maxclients，redis会给新的连接发送’max number of clients reached’，并关闭连接
# maxclients 10000
```

## memory management

```yml
#设置redis使用内存字节数，当达到内存最大值时，redis会根据选择的逐出策略尝试删除key(详细参阅maxmemory策略)#maxmemory <bytes>
#redis配置的最大内存容量。当内存满了，需要配合maxmemory-policy策略进行处理。#注意slave的输出缓冲区是不计算在maxmemory内的。所以为了防止主机内存使用完，建议设置的maxmemory需要更小一些maxmemory 122000000
#内存容量超过maxmemory后的处理策略。
#volatile-lru：利用LRU算法移除设置过过期时间的key。
#volatile-random：随机移除设置过过期时间的key。
#volatile-ttl：移除即将过期的key，根据最近过期时间来删除（辅以TTL）
#allkeys-lru：利用LRU算法移除任何key。
#allkeys-random：随机移除任何key。
#noeviction：不移除任何key，只是返回一个写错误。
#上面的这些驱逐策略，如果redis没有合适的key驱逐，对于写命令，还是会返回错误。#并且redis将不再接收写请求，只接收get请求。#写命令包括：set setnx setex append incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd # sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby zunionstore zinterstore hset # hsetnx hmset hincrby incrby decrby getset mset msetnx exec sort。
# maxmemory-policy noeviction
# lru检测的样本数。使用lru或者ttl淘汰算法，从需要淘汰的列表中随机选择sample个key，选出闲置时间最长的key移除
# maxmemory-samples 5
# 是否开启salve的最大内存
# replica-ignore-maxmemory yes
```

## lazy freeing

```yml
#以非阻塞方式释放内存
#使用以下配置指令调用了
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no
```

## 详细[全部配置](https://www.cnblogs.com/nhdlb/p/14048083.html)