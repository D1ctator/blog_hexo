---
title: redis.conf文件详细介绍
date: 2019-03-13 13:41:59
categories:
 - Redis
tags:
- Redis
---

redis.conf的一些笔记,下面整理出redis.conf中常见的一些配置介绍。<!-- more -->


```bash
# redis 主配置文件
# 基于redis 2.6.10
# 请使用 redis-server 配置文件路径 来启动redis

# 请注意一下配置文件的数据单位，大小写不敏感
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes

################################################################################
#                               redis 的基础配置                                 #
################################################################################

# 是否作为守护进程运行，生产环境用yes
daemonize yes

# 如果作为守护进程运行的话，redis会把pid打印到这个文件
# 主要多实例的时候需要写成不同的文件
pidfile /var/run/redis.pid

# redis监听的端口，注意多实例的情况
port 6379

# 允许访问redis的ip
# 测试环境注释该选项，生产环境把所有允许访问的ip都打一次
# bind xxx.xxx.xxx.xxx

# 关闭无消息的客户端的间隔，0为关闭该功能
timeout 0

# 对客户端发送ACK信息，linux中单位为秒
tcp-keepalive 0

# 数据库的数量，我们的游戏建议为1，然后多开实例
databases 1

################################################################################
#                              redis 的持久化配置                                #
################################################################################

# save 间隔 最小更新操作
# 900秒（15分钟）之后，且至少1次变更
# 300秒（5分钟）之后，且至少10次变更
# 60秒之后，且至少10000次变更
# 如果完全作为缓存开启把save全删了

# save 900 1
# save 300 10
# save 60 2000

# 持久化失败以后，redis是否停止
stop-writes-on-bgsave-error no

# 持久化的时候是否运行对字符串对象进行压缩，算法为LZF
rdbcompression yes

# 文件末尾是否包含一个CRC64的校验和
rdbchecksum yes

# redis存储数据的文件，注意多实例的时候该不同名字或者用不同的工作目录
dbfilename dump.rdb

# redis的工作目录，注意多实例的时候该不同名字或者用不同的工作目录
# 建议用不同的工作目录
dir /temp/

#设置该数据库为其他数据库的从数据库
# slaveof <masterip> <masterport>

################################################################################
#                                 redis 的限制                                  #
################################################################################

# requirepass设置客户端连接后进行任何其他指定前需要使用的密码。
# 警告:redis速度相当快,一个外部的用户可以在一秒钟进行150K次的密码尝试,
# 你需要指定非常非常强大的密码来防止暴力破解。
# requirepass foobared


# 指定与主数据库连接时需要的密码验证
# masterauth <master-password>

# 设置最多同时连接客户端数量。
# 默认没有限制，这个关系到Redis进程能够打开的文件描述符数量。
# 特殊值"0"表示没有限制。
# 一旦达到这个限制，Redis会关闭所有新连接并发送错误"达到最大用户数上限（max number of 
# clients reached）"
#
# maxclients 128

# 不要用比设置的上限更多的内存。一旦内存使用达到上限，Redis会根据选定的回收策略
#（参见：maxmemmory-policy）删除key。

# 当内存达到设置上限时，内存的淘汰策略
# maxmemmory-policy [volatile-lru or volatile-tt or volatile-randowm or allkeys-lru or allkeys-random] 

#
# 如果因为删除策略问题Redis无法删除key，或者策略设置为 "noeviction"，Redis会回复需要更多内存
# 的错误信息给命令。
# 例如，SET,LPUSH等等。但是会继续合理响应只读命令，比如：GET。
#
# 在使用Redis作为LRU缓存，或者为实例设置了硬性内存限制的时候（使用 "noeviction" 策略）的时候，
# 这个选项还是满有用的。
#
# 警告：当一堆slave连上达到内存上限的实例的时候，响应slave需要的输出缓存所需内存不计算在使用内存
# 当中。
# 这样当请求一个删除掉的key的时候就不会触发网络问题／重新同步的事件，然后slave就会收到一堆删除指
# 令，直到数据库空了为止。
#
# 简而言之，如果你有slave连上一个master的话，那建议你把master内存限制设小点儿，确保有足够的系统
# 内存用作输出缓存。
# （如果策略设置为"noeviction"的话就不无所谓了）
#
maxmemory 2gb

# 内存策略：如果达到内存限制了，Redis如何删除key。你可以在下面五个策略里面选：
# 
# volatile-lru -> 根据LRU算法生成的过期时间来删除。
# allkeys-lru -> 根据LRU算法删除任何key。
# volatile-random -> 根据过期设置来随机删除key。
# allkeys->random -> 无差别随机删。
# volatile-ttl -> 根据最近过期时间来删除（辅以TTL）
# noeviction -> 谁也不删，直接在写操作时返回错误。
# 
# 注意：对所有策略来说，如果Redis找不到合适的可以删除的key都会在写操作时返回一个错误。
#
#     这里涉及的命令：set setnx setex append
#     incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
#     sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
#     zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
#     getset mset msetnx exec sort
#
# 默认值如下：
#
# maxmemory-policy volatile-lru

maxmemory-policy allkeys-lru

# LRU和最小TTL算法的实现都不是很精确，但是很接近（为了省内存），所以你可以用样例做测试。
# 例如：默认Redis会检查三个key然后取最旧的那个，你可以通过下面的配置项来设置样本的个数。
#
# maxmemory-samples 3


# 是否开启虚拟内存支持。
# 因为 redis 是一个内存数据库,而且当内存满的时候,无法接收新的写请求,所以在redis2.0中,提供了虚拟内存的支持。
# 但是需要注意的是,redis中,所有的key都会放在内存中,在内存不够时,只会把value 值放入交换区。
# 这样保证了虽然使用虚拟内存,但性能基本不受影响,
# 同时,你需要注意的是你要把vm-max-memory设置到足够来放下你的所有的key
vm-enabled no
# vm-enabled yes


# 设置虚拟内存的交换文件路径
vm-swap-file /tmp/redis.swap

# 这里设置开启虚拟内存之后,redis将使用的最大物理内存的大小。
# 默认为0,redis将把他所有的能放到交换文件的都放到交换文件中,以尽量少的使用物理内存。
# 在生产环境下,需要根据实际情况设置该值,最好不要使用默认的 0
vm-max-memory 0

# 设置虚拟内存的页大小,如果你的 value 值比较大,比如说你要在 value 中放置博客、新闻之类的所有文章内容,就设大一点,如果要放置的都是很小的内容,那就设小一点
vm-page-size 32

# 设置交换文件的总的 page 数量,需要注意的是,page table信息会放在物理内存中,每8个page 就会占据RAM中的 1 个 byte。
# 总的虚拟内存大小 = vm-page-size * vm-pages
vm-pages 134217728

# 设置 VM IO 同时使用的线程数量。
vm-max-threads 4

################################################################################
#                               redis 的累加模式                                 #
################################################################################

# 默认情况下，Redis是异步的把数据导出到磁盘上。这种情况下，当Redis挂掉的时候，最新的数据就丢了。
# 如果不希望丢掉任何一条数据的话就该用纯累加模式：一旦开启这个模式，Redis会把每次写入的数据在接收
# 后都写入 appendonly.aof 文件。
# 每次启动时Redis都会把这个文件的数据读入内存里。
#
# 注意，异步导出的数据库文件和纯累加文件可以并存（你得把上面所有"save"设置都注释掉，关掉导出机制）。
# 如果纯累加模式开启了，那么Redis会在启动时载入日志文件而忽略导出的 dump.rdb 文件。
#
# 重要：查看 BGREWRITEAOF 来了解当累加日志文件太大了之后，怎么在后台重新处理这个日志文件。
appendonly no

# 纯累加文件名字（默认："appendonly.aof"）
# appendfilename appendonly.aof

# 纯累加文件的flush频率
# always    ->  每次写入都flush，最安全，资源开销最大
# everysec  ->  每秒 (推荐)
# no        ->  由系统确定

# appendfsync always
appendfsync everysec
# appendfsync no

# 当纯累加文件进行rewrite时，是否需要fsync
# 当且仅当appendfsync = always || everysec 时该参数生效
no-appendfsync-on-rewrite no

# 纯累加文件下次rewrite的比例，与纯累加文件文件的最小size
# 下面的参数意味着纯累加文件会在512mb的时候进行一次rewrite
# 若rewrite后的文件大小为x mb，则下次纯累加文件将会在2x mb时rewrite
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 512mb

################################################################################
#                                redis 的高级配置                                #
################################################################################

# redis 2.0 中引入了 hash 数据结构。 
# 如果hash中的数量超出hash-max-ziplist-entries，或者value的长度超出
# hash-max-ziplist-value，将改成保存dict，否则以ziphash的方式存储以节省空间。以下同理。
# hash 中包含超过指定元素个数并且最大的元素当没有超过临界时,hash 将以zipmap(又称为 small hash大大减少内存使用)来存储,这里可以设置这两个临界值
hash-max-ziplist-entries 64
hash-max-ziplist-value 128

list-max-ziplist-entries 64
list-max-ziplist-value 128

set-max-intset-entries 64

zset-max-ziplist-entries 64
zset-max-ziplist-value 128

# 是否resize hash? 如果你设置成no需要在源码做一定的修改以防止有人进行hash攻击
# 开启之后,redis 将在每 100 毫秒时使用 1 毫秒的 CPU 时间来对 redis 的 hash 表进行重新 hash,可以降低内存的使用。
# 当你的使用场景中,有非常严格的实时性需要,不能够接受 Redis 时不时的对请求有 2 毫秒的延迟的话,把这项配置为 no。
# 如果没有这么严格的实时性要求,可以设置为 yes,以便能够尽可能快的释放内存
activerehashing yes

################################################################################
#                              redis 的其他配置文件                               #
################################################################################

# 日志配置
include /etc/redis/log.conf

# 主从配置
# include /etc/redis/slave.conf

# 安全配置
# include /etc/redis/security.conf

# LUA配置
# include /etc/redis/lua.conf

```
