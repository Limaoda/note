#### 检查是否有yum源

```shell
yum install redis
```

#### 下载fedora的epel仓库

```shell
yum install epel-release
```

#### 安装redis数据库

```shell
yum install redis
```

#### 安装完毕后，使用下面的命令启动redis服务

```shell
# 启动redis
service redis start
# 停止redis
service redis stop
# 查看redis运行状态
service redis status
# 查看redis进程
ps -ef | grep redis
```

#### 设置redis为开机自动启动

```redis
chkconfig redis on
```

#### 进入redis服务

~~~shell
# 进入本机redis
redis-cli
# 列出所有key
keys *
``
- 防火墙开放相应端口
```bash
# 开启6379
/sbin/iptables -I INPUT -p tcp --dport 6379 -j ACCEPT
# 开启6380
/sbin/iptables -I INPUT -p tcp --dport 6380 -j ACCEPT
# 保存
/etc/rc.d/init.d/iptables save
# centos 7下执行
service iptables save
``

### 修改redis默认端口和密码
- 打开配置文件
```bash
vi /etc/redis.conf
~~~

#### 修改默认端口，查找 port 6379 修改为相应端口即可

![](C:\Users\HuQiaoDong\Desktop\笔记\images\port_redis.png)

#### 修改默认密码，查找 requirepass foobared 将 foobared 修改为你的密码

![](C:\Users\HuQiaoDong\Desktop\笔记\images\pass_redis.png)

 #### 使用配置文件启动 redis

```shell
redis-server /etc/redis.conf &
```

#### 使用端口登录

```shell
redis-cli -h 127.0.0.1 -p 6179
```

#### 输入刚才输入的密码

```shell
auth 111
```

![](C:\Users\HuQiaoDong\Desktop\笔记\images\pass.png)

#### 停止redis

```shell
redis-cli -h 127.0.0.1 -p 6179
shutdown
```

#### 进程号杀掉redis

```shell
ps -ef | grep redis
kill -9 XXX
```

#### 使用redis desktop manager远程连接redis

如果长时间连接不上，可能有两种可能性    

1、bind了127.0.01：只允许在本机连接redis  

 2、protected-mode设置了yes（使用redis desktop manager工具需要配置，其余不用）

**解决办法：**

```shell
# 打开redis配置文件
vi /etc/redis.conf
# 找到 bind 127.0.0.1 将其注释
# 找到 protected-mode yes 将其改为
protected-mode no
```

#### 重启redis

```shell
service redis stop
service redis start
```

