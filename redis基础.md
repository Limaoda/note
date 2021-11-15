#### 性能测试(benchmark)

```shell
cd /bin
#测试并发100个连接，每个连接发起100000个请求
redis-benchmark -h localhost -p 6379 -c 100 -n 100000
```

#### 基本操作

```bash
# 切换数据库，redis默认有16个数据库[0-15],默认使用0
127.0.0.1:6379> select [index]

# 查看数据库大小
127.0.0.1:6379> dbsize

# 设置Redis配置
127.0.0.1:6379> config set [option] [value]

# 读取Redis配置
127.0.0.1:6379> config get [option] [value]

# 保存配置信息至配置文件中
127.0.0.1:6379> save

# 查看当前数据库所有的key
127.0.0.1:6379> keys *

# 查看当前数据库的key个数
127.0.0.1:6379> dbsize

# 随机返回当前数据库中的一个key
127.0.0.1:6379> randomkey

# 清空当前数据库
127.0.0.1:6379> flushdb

# 清空所有数据库数据
127.0.0.1:6379> flushall

# 设置key以及对应的value
127.0.0.1:6379> set [key] [value]

# 查看某个key对应的value数据类型
127.0.0.1:6379> type [key]

# 给某个key添加过期时间，可用于缓存
127.0.0.1:6379> expire [key] [seconds]

# 查看某个key存在的时效值
127.0.0.1:6379> ttl [key]

# 获取某个key对应的value
127.0.0.1:6379> get [key]

# 判断是否存在某个key
127.0.0.1:6379> exists [key]

# 重命名某个key
127.0.0.1:6379> rename [key] [name]

# 移动某个key到指定数据库
127.0.0.1:6379> move [key] [db_index]

# 删除某个key
127.0.0.1:6379> del [key]
```

#### 危险操作

```bash
# 单行遍历，速度很慢很占时间，单核CPU情况下数据量过多时可能因为CPU处理不过来导致宕机
keys *

# 清空当前库
flushdb 

# 清空所有库
flushall

# 修改配置文件
config
```



#### 基础知识

* **Redis可以用作数据库、缓存和消息中间件**

* **6.0版本之前，Redis是单线程的，CPU不是Redis的瓶颈，服务器内存和网络带宽才是**
* **Redis是C语言编写的**

* **Redis基本所有数据全部存放在内存中，因此单线程操作效率最高，多线程会进行CPU上下文切换，非常耗时，因此对于一个内存系统，单线程操作效率最高，多次读写全部在一个CPU上进行（CPU上下文切换一次大概耗费1500~2000纳秒）**
* **Redis使用了IO多路复用模型**

#### 数据类型

* **string（字符串）**

  string 是 redis **最基本**的类型，一个 key 对应一个 value。

  string 类型是二进制安全的。意思是 redis 的 string 可以包含任何数据。比如jpg图片或者序列化的对象。

  string 类型是 Redis 最基本的数据类型，string 类型的值最大能存储 512MB。

  ```sql
  # 向某个key对应的value（字符串类型）追加字符串,当不存在已有key时，append相当于set命令
  append [key] "http"
  
  # 查看某个key对应的value字符串长度
  strlen [key] # [integer]length
  
  # 对某个值为数值的value进行自增
  incr [key]
  
  # 对某个值为数值的value进行自减
  decr [key]
  
  # 对某个值为数值的value进行定长自增
  incrby [key] [length]
  
  # 对某个值为数值的value进行定长自减
  decrby [key] [length]
  
  # 获取某个key对应的value（字符串类型）并进行子字符串截取,相当于java的substring方法
  getrange [key] [start] [end]
  
  # 对某个key对应的value（字符串类型）进行子字符串替换，相当于java的replace
  setrange [key] [offset] [value]
  
  # 设置具有过期时间的key
  setex [key] [seconds] [value]
  
  # 判断是否存在key，不存在则进行设置,否则无效（分布式锁中常用）
  setnx [key] [value]
  
  # 批量插入数据
  mset [key] [value] [key] [value] ...
  
  # 判断是否存在key并批量插入数据（！注意，有一个key为存在，则全部为失败，属于原子性操作）
  msetnx [key] [value] [key] [value] ...
  
  # 批量获取数据
  mget [key] [key] [key] ...
  
  # 插入获取组合命令
  getset [key] [value]
  
  ```
  以B站为例，现在我们要对某个b站用户某个视频的点赞量、投币量、收藏量进行统计，可以这样设计key

```bash
uuid:1231:video:1:star #uuid为1231的用户的id为1的视频的收藏量
uuid:1231:video:1:good #uuid为1231的用户的id为1的视频的点赞量
uuid:1231:video:1:coin #uuid为1231的用户的id为1的视频的投币量

#插入数据
mset uuid:1231:video:1:star 0 uuid:1231:video:1:good 0 uuid:1231:video:1:coin 0

#一键三连
incr uuid:1231:video:1:star
incr uuid:1231:video:1:good
incr uuid:1231:video:1:coin
```


  > **注意：**string类型的值一个键最大能存储 512MB。




* **list（列表）**

  list可以实现栈、队列、阻塞队列

  Redis的list相关命令都是以l开头的，除了push和pop的l和r代表的是左右，而不是list

  ```bash
  # 将一个或者多个值从左边插入
  lpush [key] [value] [value] ...
  
  # 将一个或者多个值从右边插入
  rpush [key] [value] [value] ...
  
  # 将最左边的元素移出列表
  lpop [key]
  
  # 将最右边的元素移出列表
  rpop [key]
  
  # 截取一部分元素作为子列表返回，当start为0，end为-1时取出全部元素
  lrange [key] [start] [end]
  
  # 获取某个列表下标索引对应的值，index为负值时，从最右边开始往左计算索引
  lindex [key] [index]
  
  # 删除列表中一个或多个指定值的元素,count>0从左往右移除，count=0移除全部,count<0从右往左移除
  lrem [key] [count] [value]
  
  # 截取列表，该操作改变原列表，返回截取的子列表
  ltrim [key] [start] [end]
  
  # 组合操作，将列表的最右边元素弹出并从指定列表的左端插入
  rpoplpush [key1] [key2] 
  
  # 重新设置列表某个下标索引对应的值
  lset [key] [index] [value]
  
  # 插入新值到指定值的元素前后位置
  linsert [key] [before/after] [value] [new_value] 
  
  # 获取列表长度
  llen [key]
  ```

* **hash（哈希）**

  哈希类型的结构为key-{key-value},实际上，原本的value在哈希类型中就是一个map，存放了多个键值对

  Redis的哈希类型相关命令都以h开头

  ```bash
  # 插入某个key和对应的哈希键值对
  hset [key] [field] [value]
  
  # 获取某个key和对应属性的值
  hget [key] [field]
  
  # 批量插入
  hmset [key] [field] [value] [field] [value] ...
  
  # 批量获取
  hmget [key] [field] [field] ...
  
  # 删除某个key对应的哈希键值对
  hdel [key] [field]
  
  # 获取某个key对应的所有键值对
  hgetall [key]
  
  # 获取某个key对应的键值对个数（长度）
  hlen [key]
  
  # 获取某个key对应的所有哈希键
  hkeys [key]
  
  # 获取某个key对应的所有哈希值
  hvals [key]
  
  # 判断某个key对应的哈希键值对中是否存在某个键
  hexists [key]
  
  # 数值自增自减，不同于String类型，hash中没有incr、decr、decrby命令，当increment为负值时相当于decrby自减
  hincrby [key] [field] [increment]
  
  #与String类型的setnx一致。存在键则失效，不存在键则创建
  hsetnx [key] [field] [value]
  ```
  
* **set（集合）**

  set的特点是无序不可重复

  Redis的集合类型相关命令都以s开头

  set的交并差集合运算可以用来实现共同关注好友等功能

  ```bash
  # 往集合中添加成员
  sadd [key] [member]
  
  # 查看集合中的所有成员
  smembers [key]
  
  # 判断某一个值是否在set集合中
  sismember [key] [member]
  
  # 查看集合的成员个数
  scard [key]
  
  # 移除集合中一个或多个成员
  srem [key] [member] [member] ...
  
  # 随机抽选出指定数量成员
  srandmember [key] [count]
  
  # 随机移除集合中的一个或多个成员
  spop [key] [count]
  
  # 移动集合中的某个元素到另一个集合中
  smove [key1] [key2] [member]
  
  
  
  ###########数学集合#########
  # 差集，找出key2集合中存在，key1集合中不存在的成员集
  sdiff [key1] [key2]
  
  # 交集，找出key1集合与key2集合中同时存在的成员集
  sinter [key1] [key2]
  
  # 并集，找出key1集合与key2集合所有成员合并去重后的成员集合
  sunion [key1] [key2]
  
  ```

  

* **zset(sorted set：有序集合)**

  zset的score可以用于权重标识，用来进行权重判断

  zset可以用于排行榜应用

  ```bash
  # 向有序集合
  zadd [key] [score] [member]
  
  # 获取数据
  zrange [key] [start] [end]
  
  # 移除数据
  zrem [key] [member]
  
  # 范围排序（升序）,min可以为-inf（负无穷）或score，max可以为+inf（正无穷）或score
  zrangebyscore [key] [min] [max] [withscores]
  
  # 范围排序（降序）,min可以为-inf（负无穷）或score，max可以为+inf（正无穷）或score
  zrevrangebyscore [key] [max] [min] [withscores]
  
  # 获取有序集合中所有成员数量
  zcard [key] 
  
  # 获取指定区间的成员数量
  zcount [key] [start] [end]
  
  # 对zset某个成员的score进行自增,
  zincrby [key] [increment] [member]
  
  # 对zset某个成员的score进行自减
  zdecrby [key] [increment] [member]
  ```
  
  
#### Redis事务

##### Redis事务概述

Redis中的命令具有原子性（要么同时成功要么同时失败），事务不具有原子性和隔离性（单线程，不需要隔离，需要手动执行）

Redis事务性质：一次性（所有命令一次性执行）、顺序性（事务中的命令顺序执行）、排他性

>  事务的本质：一组命令的集合

```bash
# 开启Redis事务
multi

# 事务中的所有命令都会进入事务队列，不执行
set [key] [value] # Queued
get [key] [value] # Queued

# 执行Redis事务，执行完毕事务自动销毁，生命周期结束
exec

# 放弃事务，事务销毁，生命周期结束（需要存在已经开启的事务）
discard
```

事务中存在两类异常，编译型异常和运行时异常，编译型异常出现时，整个事务的所有操作全部失败，运行时异常则只是出现异常的那一个redis操作失败，事务中其它的redis操作不受影响

```bash
multi

set name Bob

set age 20

getset name #这里缺少value

exec # 报错，属于编译型异常，整个事务的所有命令不执行

multi

set key "aa"

incr key

get key

exec # 只有incr key这个操作报错执行失败（运行时异常），其它操作执行成功
```

##### Redis乐观锁的实现

```bash
# 银行转账场景，Bob账户里有100块钱，Linda账户里有10块钱，Bob转20块钱给Linda，事务结束，Bob账户应该剩80块钱，Linda账户应该有30块钱,但在这个事务执行中途Linda收到了她爸爸转给她的100块钱，此时事务需要作失败处理，否则Linda的账户余额会混乱
set Bob:money 100

set Linda:money 10

watch Linda:money #监测Linda账户余额，如果在事务执行途中发生变化，则事务执行失败（watch实际上也是将key进行保存，当事务被执行时会将key的状态与保存的key副本进行对比，如果有不一致的地方则放弃事务）

# 开启事务
multi

decrby Bob:money 20

incrby Linda:money 20

# 此时Linda的爸爸转给Linda 100块钱
# incrby Linda:money 100

exec # 事务执行失败，此时Bob的账户余额为100，Linda的账户为110，无论事务是否执行成功，都会自动执行unwatch进行解锁
```

> 乐观锁与悲观锁都是并发问题中的概念，乐观锁指的是当进行某一个操作时，默认不会被其它并发操作影响到本次操作，因此不会上锁，在mysql中我们通常给表加一个version字段，每次操作这条数据时，就进行一次version自增，当mysql事务执行时会拿到这条数据的version与执行前进行对比，如果发现version被修改，那么事务失败。悲观锁指的是当进行某一个操作时，默认会被其它并发操作影响到本次操作，因此在操作时会将数据锁住不允许修改，这时任何其它并发操作都会失败（线程阻塞），等到操作结束后拿到锁才允许其它并发线程访问。乐观锁因为不阻塞线程，因此在性能上比悲观锁要好得多。

#### Redis持久化

RDB（==R==edis ==D==ata==B==ase），在进行RDB时，Redis服务器的父进程会fork出一个子进程，子进程将所有内存中的Redis存储数据全部写入到RDB临时文件中，这个过程不会影响到Redis父进程的服务，临时文件写入完成后，会替换掉上一次的正式RDB文件，此时子进程退出。Redis默认的持久化方式就是RDB，因此不需要额外在配置文件中进行配置。redis保存的文件是dump.rdb

RDB的自动触发机制

1.满足save规则

2.执行flushall命令

3.退出redis或执行shutdown（默认进行save）

优点：

适合大规模的数据恢复

对数据的完整性要求不高

缺点：

需要在一段时间内连续进行操作才会触发，意外宕机时，即使save 1 1，最后一条数据仍可能无法保存

fork的子进程会占用一定的内存

> 在生产环境中，有时会对dump.rdb文件进行备份

#### Redis配置文件详解

* redis配置文件对大小写不敏感

redis.conf

##### 一些基本内存单位的换算

```bash
# Note on units: when memory size is needed, it is possible to specify
# it in the usual form of 1k 5GB 4M and so forth:
#
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# units are case insensitive so 1GB 1Gb 1gB are all the same.
```

##### 多个redis配置文件组合使用

```bash
# Include one or more other config files here.  This is useful if you
# have a standard template that goes to all Redis servers but also need
# to customize a few per-server settings.  Include files can include
# other files, so use this wisely.
#
# Notice option "include" won't be rewritten by command "CONFIG REWRITE"
# from admin or Redis Sentinel. Since Redis always uses the last processed
# line as value of a configuration directive, you'd better put includes
# at the beginning of this file to avoid overwriting config change at runtime.
#
# If instead you are interested in using includes to override configuration
# options, it is better to use include as the last line.
#
# include /path/to/local.conf
# include /path/to/other.conf
```

##### 网络配置

 ```bash
 # By default, if no "bind" configuration directive is specified, Redis listens
 # for connections from all the network interfaces available on the server.
 # It is possible to listen to just one or multiple selected interfaces using
 # the "bind" configuration directive, followed by one or more IP addresses.
 #
 # Examples:
 #
 # bind 192.168.1.100 10.0.0.1
 # bind 127.0.0.1 ::1
 #
 # ~~~ WARNING ~~~ If the computer running Redis is directly exposed to the
 # internet, binding to all the interfaces is dangerous and will expose the
 # instance to everybody on the internet. So by default we uncomment the
 # following bind directive, that will force Redis to listen only into
 # the IPv4 lookback interface address (this means Redis will be able to
 # accept connections only from clients running into the same computer it
 # is running).
 #
 # IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
 # JUST COMMENT THE FOLLOWING LINE.
 # ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 bind 127.0.0.1 # 绑定的ip，这里redis默认绑定本地ip，因此只能本机使用，如果需要远程连接，那么可以将ip更换为*进行通配或者使用0.0.0.0
 
 # Protected mode is a layer of security protection, in order to avoid that
 # Redis instances left open on the internet are accessed and exploited.
 #
 # When protected mode is on and if:
 #
 # 1) The server is not binding explicitly to a set of addresses using the
 #    "bind" directive.
 # 2) No password is configured.
 #
 # The server only accepts connections from clients connecting from the
 # IPv4 and IPv6 loopback addresses 127.0.0.1 and ::1, and from Unix domain
 # sockets.
 #
 # By default protected mode is enabled. You should disable it only if
 # you are sure you want clients from other hosts to connect to Redis
 # even if no authentication is configured, nor a specific set of interfaces
 # are explicitly listed using the "bind" directive.
 protected-mode yes # 是否为受保护模式，一般为开启状态，开启状态下无法远程连接
 
 # Accept connections on the specified port, default is 6379 (IANA #815344).
 # If port 0 is specified Redis will not listen on a TCP socket.
 port 6379 # 端口设置，redis集群需要修改端口，从安全性上考虑，也不应该使用6379默认端口
 
 # TCP listen() backlog.
 #
 # In high requests-per-second environments you need an high backlog in order
 # to avoid slow clients connections issues. Note that the Linux kernel
 # will silently truncate it to the value of /proc/sys/net/core/somaxconn so
 # make sure to raise both the value of somaxconn and tcp_max_syn_backlog
 # in order to get the desired effect.
 tcp-backlog 511
 
 # Unix socket.
 #
 # Specify the path for the Unix socket that will be used to listen for
 # incoming connections. There is no default, so Redis will not listen
 # on a unix socket when not specified.
 #
 # unixsocket /tmp/redis.sock
 # unixsocketperm 700
 
 # Close the connection after a client is idle for N seconds (0 to disable)
 timeout 0
 
 # TCP keepalive.
 #
 # If non-zero, use SO_KEEPALIVE to send TCP ACKs to clients in absence
 # of communication. This is useful for two reasons:
 #
 # 1) Detect dead peers.
 # 2) Take the connection alive from the point of view of network
 #    equipment in the middle.
 #
 # On Linux, the specified value (in seconds) is the period used to send ACKs.
 # Note that to close the connection the double of the time is needed.
 # On other kernels the period depends on the kernel configuration.
 #
 # A reasonable value for this option is 300 seconds, which is the new
 # Redis default starting with Redis 3.2.1.
 tcp-keepalive 300
 ```

##### 通用配置

```bash
# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
daemonize no # 是否以守护进程（后台）的方式运行，默认不开启

# If you run Redis from upstart or systemd, Redis can interact with your
# supervision tree. Options:
#   supervised no      - no supervision interaction
#   supervised upstart - signal upstart by putting Redis into SIGSTOP mode
#   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET
#   supervised auto    - detect upstart or systemd method based on
#                        UPSTART_JOB or NOTIFY_SOCKET environment variables
# Note: these supervision methods only signal "process is ready."
#       They do not enable continuous liveness pings back to your supervisor.
supervised no

# If a pid file is specified, Redis writes it where specified at startup
# and removes it at exit.
#
# When the server runs non daemonized, no pid file is created if none is
# specified in the configuration. When the server is daemonized, the pid file
# is used even if not specified, defaulting to "/var/run/redis.pid".
#
# Creating a pid file is best effort: if Redis is not able to create it
# nothing bad happens, the server will start and run normally.
pidfile /var/run/redis_6379.pid # 如果使用守护进程方式运行，需要指定一个pid文件

# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing) 用于测试和开发阶段
# verbose (many rarely useful info, but not a mess like the debug level) 记录较多日志信息，类似debug
# notice (moderately verbose, what you want in production probably) 生产环境推荐使用，只包含部分重要信息
# warning (only very important / critical messages are logged) 只有十分重要和关键的信息才会保存日志
loglevel notice # 日志级别

# Specify the log file name. Also the empty string can be used to force
# Redis to log on the standard output. Note that if you use standard
# output for logging but daemonize, logs will be sent to /dev/null
logfile /var/log/redis/redis.log # 生成的日志文件存放位置

# To enable logging to the system logger, just set 'syslog-enabled' to yes,
# and optionally update the other syslog parameters to suit your needs.
# syslog-enabled no

# Specify the syslog identity.
# syslog-ident redis

# Specify the syslog facility. Must be USER or between LOCAL0-LOCAL7.
# syslog-facility local0

# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
databases 16 # 数据库数量，默认为16个
```

##### 快照（持久化）

> redis是个内存型数据库，如果没有持久化，断电会丢失数据，快照可以防止数据丢失

```bash
# Save the DB on disk:
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
#
#   In the example below the behaviour will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   Note: you can disable saving completely by commenting out all "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
#
#   save ""

# 如果900秒内，有至少1个key进行了修改操作，则进行持久化（触发rdb）
save 900 1
# 如果300秒内，有至少10个key进行了修改操作，则进行持久化（触发rdb）
save 300 10
# 如果60秒内，有至少10000个key进行了修改操作，则进行持久化（触发rdb）
save 60 10000

# By default Redis will stop accepting writes if RDB snapshots are enabled
# (at least one save point) and the latest background save failed.
# This will make the user aware (in a hard way) that data is not persisting
# on disk properly, otherwise chances are that no one will notice and some
# disaster will happen.
#
# If the background saving process will start working again Redis will
# automatically allow writes again.
#
# However if you have setup your proper monitoring of the Redis server
# and persistence, you may want to disable this feature so that Redis will
# continue to work as usual even if there are problems with disk,
# permissions, and so forth.
stop-writes-on-bgsave-error yes # 持久化发生错误时，是否停止写入，默认开启

# Compress string objects using LZF when dump .rdb databases?
# For default that's set to 'yes' as it's almost always a win.
# If you want to save some CPU in the saving child set it to 'no' but
# the dataset will likely be bigger if you have compressible values or keys.
rdbcompression yes # 是否压缩rdb文件，压缩会耗费cpu资源

# Since version 5 of RDB a CRC64 checksum is placed at the end of the file.
# This makes the format more resistant to corruption but there is a performance
# hit to pay (around 10%) when saving and loading RDB files, so you can disable it
# for maximum performances.
#
# RDB files created with checksum disabled have a checksum of zero that will
# tell the loading code to skip the check.
rdbchecksum yes # 保存rdb文件时是否校验rdb文件，如果出错自动进行修复，默认开启

# The filename where to dump the DB
dbfilename dump.rdb

# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
dir /var/lib/redis # rdb文件保存的目录
```

##### 安全

```bash
# Require clients to issue AUTH <PASSWORD> before processing any other
# commands.  This might be useful in environments in which you do not trust
# others with access to the host running redis-server.
#
# This should stay commented out for backward compatibility and because most
# people do not need auth (e.g. they run their own servers).
#
# Warning: since Redis is pretty fast an outside user can try up to
# 150k passwords per second against a good box. This means that you should
# use a very strong password otherwise it will be very easy to break.
#
requirepass foobared # 密码设置


# Command renaming.
#
# It is possible to change the name of dangerous commands in a shared
# environment. For instance the CONFIG command may be renamed into something
# hard to guess so that it will still be available for internal-use tools
# but not available for general clients.
#
# Example:
#
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
#
# It is also possible to completely kill a command by renaming it into
# an empty string:
#
# rename-command CONFIG ""
#
# Please note that changing the name of commands that are logged into the
# AOF file or transmitted to slaves may cause problems.
```

##### 限制

```bash
# Set the max number of connected clients at the same time. By default
# this limit is set to 10000 clients, however if the Redis server is not
# able to configure the process file limit to allow for the specified limit
# the max number of allowed clients is set to the current file limit
# minus 32 (as Redis reserves a few file descriptors for internal uses).
#
# Once the limit is reached Redis will close all the new connections sending
# an error 'max number of clients reached'.
#
# maxclients 10000 # redis客户端最大连接数

# Don't use more memory than the specified amount of bytes.
# When the memory limit is reached Redis will try to remove keys
# according to the eviction policy selected (see maxmemory-policy).
#
# If Redis can't remove keys according to the policy, or if the policy is
# set to 'noeviction', Redis will start to reply with errors to commands
# that would use more memory, like SET, LPUSH, and so on, and will continue
# to reply to read-only commands like GET.
#
# This option is usually useful when using Redis as an LRU cache, or to set
# a hard memory limit for an instance (using the 'noeviction' policy).
#
# WARNING: If you have slaves attached to an instance with maxmemory on,
# the size of the output buffers needed to feed the slaves are subtracted
# from the used memory count, so that network problems / resyncs will
# not trigger a loop where keys are evicted, and in turn the output
# buffer of slaves is full with DELs of keys evicted triggering the deletion
# of more keys, and so forth until the database is completely emptied.
#
# In short... if you have slaves attached it is suggested that you set a lower
# limit for maxmemory so that there is some free RAM on the system for slave
# output buffers (but this is not needed if the policy is 'noeviction').
#
# maxmemory <bytes> # redis可用的最大内存，单位为字节

# MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
# is reached. You can select among five behaviors:
#
# volatile-lru -> remove the key with an expire set using an LRU algorithm
# allkeys-lru -> remove any key according to the LRU algorithm
# volatile-random -> remove a random key with an expire set
# allkeys-random -> remove a random key, any key
# volatile-ttl -> remove the key with the nearest expire time (minor TTL)
# noeviction -> don't expire at all, just return an error on write operations
#
# Note: with any of the above policies, Redis will return an error on write
#       operations, when there are no suitable keys for eviction.
#
#       At the date of writing these commands are: set setnx setex append
#       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
#       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
#       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
#       getset mset msetnx exec sort
#
# The default is:
#
# maxmemory-policy noeviction #内存到达上限后的处理策略（淘汰命中规则的key）

# LRU and minimal TTL algorithms are not precise algorithms but approximated
# algorithms (in order to save memory), so you can tune it for speed or
# accuracy. For default Redis will check five keys and pick the one that was
# used less recently, you can change the sample size using the following
# configuration directive.
#
# The default of 5 produces good enough results. 10 Approximates very closely
# true LRU but costs a bit more CPU. 3 is very fast but not very accurate.
#
# maxmemory-samples 5
```

##### aof配置

```bash
# By default Redis asynchronously dumps the dataset on disk. This mode is
# good enough in many applications, but an issue with the Redis process or
# a power outage may result into a few minutes of writes lost (depending on
# the configured save points).
#
# The Append Only File is an alternative persistence mode that provides
# much better durability. For instance using the default data fsync policy
# (see later in the config file) Redis can lose just one second of writes in a
# dramatic event like a server power outage, or a single write if something
# wrong with the Redis process itself happens, but the operating system is
# still running correctly.
#
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
#
# Please check http://redis.io/topics/persistence for more information.

appendonly no # 默认不开启aof模式，redis默认以rdb方式进行持久化，一般情况下不会用aof

# The name of the append only file (default: "appendonly.aof")

appendfilename "appendonly.aof" # 持久化文件名

# The fsync() call tells the Operating System to actually write data on disk
# instead of waiting for more data in the output buffer. Some OS will really flush
# data on disk, some other OS will just try to do it ASAP.
#
# Redis supports three different modes:
#
# no: don't fsync, just let the OS flush the data when it wants. Faster.
# always: fsync after every write to the append only log. Slow, Safest.
# everysec: fsync only one time every second. Compromise.
#
# The default is "everysec", as that's usually the right compromise between
# speed and data safety. It's up to you to understand if you can relax this to
# "no" that will let the operating system flush the output buffer when
# it wants, for better performances (but if you can live with the idea of
# some data loss consider the default persistence mode that's snapshotting),
# or on the contrary, use "always" that's very slow but a bit safer than
# everysec.
#
# More details please check the following article:
# http://antirez.com/post/redis-persistence-demystified.html
#
# If unsure, use "everysec".

# appendfsync always # 每次修改执行一次同步，消耗性能
appendfsync everysec # 每秒执行一次同步
# appendfsync no     # 不执行同步，操作系统自己同步数据，速度最快

# When the AOF fsync policy is set to always or everysec, and a background
# saving process (a background save or AOF log background rewriting) is
# performing a lot of I/O against the disk, in some Linux configurations
# Redis may block too long on the fsync() call. Note that there is no fix for
# this currently, as even performing fsync in a different thread will block
# our synchronous write(2) call.
#
# In order to mitigate this problem it's possible to use the following option
# that will prevent fsync() from being called in the main process while a
# BGSAVE or BGREWRITEAOF is in progress.
#
# This means that while another child is saving, the durability of Redis is
# the same as "appendfsync none". In practical terms, this means that it is
# possible to lose up to 30 seconds of log in the worst scenario (with the
# default Linux settings).
#
# If you have latency problems turn this to "yes". Otherwise leave it as
# "no" that is the safest pick from the point of view of durability.

no-appendfsync-on-rewrite no

# Automatic rewrite of the append only file.
# Redis is able to automatically rewrite the log file implicitly calling
# BGREWRITEAOF when the AOF log size grows by the specified percentage.
#
# This is how it works: Redis remembers the size of the AOF file after the
# latest rewrite (if no rewrite has happened since the restart, the size of
# the AOF at startup is used).
#
# This base size is compared to the current size. If the current size is
# bigger than the specified percentage, the rewrite is triggered. Also
# you need to specify a minimal size for the AOF file to be rewritten, this
# is useful to avoid rewriting the AOF file even if the percentage increase
# is reached but it is still pretty small.
#
# Specify a percentage of zero in order to disable the automatic AOF
# rewrite feature.

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# An AOF file may be found to be truncated at the end during the Redis
# startup process, when the AOF data gets loaded back into memory.
# This may happen when the system where Redis is running
# crashes, especially when an ext4 filesystem is mounted without the
# data=ordered option (however this can't happen when Redis itself
# crashes or aborts but the operating system still works correctly).
#
# Redis can either exit with an error when this happens, or load as much
# data as possible (the default now) and start if the AOF file is found
# to be truncated at the end. The following option controls this behavior.
#
# If aof-load-truncated is set to yes, a truncated AOF file is loaded and
# the Redis server starts emitting a log to inform the user of the event.
# Otherwise if the option is set to no, the server aborts with an error
# and refuses to start. When the option is set to no, the user requires
# to fix the AOF file using the "redis-check-aof" utility before to restart
# the server.
#
# Note that if the AOF file will be found to be corrupted in the middle
# the server will still exit with an error. This option only applies when
# Redis will try to read more data from the AOF file but not enough bytes
# will be found.
aof-load-truncated yes
```





#### Redis安全机制 

##### 开启redis密码，并设置高复杂度密码

方式一：更改配置文件(重启生效)

```bash
# 使用sha256算法生成加密字符串
echo "123456" | sha256sum

# 编辑redis配置文件
vim /etc/redis.conf

# 找到以下行，将#去掉，foobared改成自己的密码
# requirepass foobared

# 重启服务
redis-server /etc/redis.conf
```



方式二：客户端进行密码设置（不重启生效）

> 如果配置文件中没添加密码 那么redis重启后，密码失效；

```bash
# 连接redis客户端
redis-cli

# 使用客户端设置密码
127.0.0.1:6379> config set requirepass $password

# 查询密码
127.0.0.1:6379> config get requirepass #$password

# 密码验证
127.0.0.1:6379> auth $password #ok
```

登陆有密码的Redis

```bash
# 登录时输入密码
redis-cli -p $port -a $password

# 登陆后验证密码
redis-cli -p $port

# 验证密码
127.0.0.1:6379> auth $password
```

##### 不要使用默认端口

6379是redis的默认端口号，尽量修改为自定义的端口号，这样即便被别人知道ip和密码，对方不知道端口号也无法访问

```bash
# redis.conf文件
port 6379 # 改为自定义的端口号

# port 6700
```

##### 禁止root用户启动

为 Redis 服务创建单独的用户和home目录，使用普通用户启动，安全性往往高很多；
业务程序永久别用root用户运行。

##### 在信任的内网运行，尽量避免有公网访问



##### 限制redis文件目录访问权限



##### 禁止任意地址连接

```bash
# redis.conf文件
bind 0.0.0.0 # 改为允许连接的ip白名单

# bind 127.0.0.1 10.254.8.4
```

##### 禁用或重命名危险命令

```bash
 # redis.conf文件
 rename-command FLUSHALL "" # 禁用清空所有数据库命令
 rename-command FLUSHDB "" # 禁用清空当前数据库命令
 rename-command KEYS "" 
 rename-command CONFIG "" # 禁用配置相关命令
```







 

