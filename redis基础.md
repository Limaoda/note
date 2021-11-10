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

# 查看数据库所有的key
127.0.0.1:6379> keys *

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

# 移动某个key到指定数据库
127.0.0.1:6379> move [key] [db_index]

# 删除某个key
127.0.0.1:6379> del [key]
```

#### 基础知识

* **Redis可以用作数据库、缓存和消息中间件**

* **Redis是单线程的，CPU不是Redis的瓶颈，服务器内存和网络带宽才是**
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
  
  
  ```

  
#### Redis事务

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

##### 登陆有密码的Redis

```bash
# 登录时输入密码
redis-cli -p $port -a $password

# 登陆后验证密码
redis-cli -p $port

# 验证密码
127.0.0.1:6379> auth $password
```



禁用或重命名危险命令
