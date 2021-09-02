# Mysql安装

----

## 1. 卸载

```shell
rpm -e --nodeps $(rpm -qa | grep mariadb)
```



## 2. 安装

### 2.1 下载

官网下载：https://dev.mysql.com/downloads/mysql/

这里以linux通用版mysql-8.0.18为例：下载方式：

```shell
# 注意，如果你使用的DNS是美国的8.8.8.8或8.8.4.4，下载可能会很慢或直接链接失败，建议把DNS改为114.114.114.114
wget https://downloads.mysql.com/archives/get/p/23/file/mysql-8.0.18-linux-glibc2.12-x86_64.tar.xz
```



### 2.2 解压

```shell
tar Jxvf mysql-8.0.18-linux-glibc2.12-x86_64.tar.xz -C /usr/local/
```



### 2.3 重命名文件夹

```shell
mv /usr/local/mysql-8.0.18-linux-glibc2.12-x86_64 /usr/local/mysql
```



### 2.4 环境变量

```shell
#添加环境变量配置
echo -e "\n\n#Mysql\nexport MYSQL_HOME=/usr/local/mysql\nexport PATH=.:$MYSQL_HOME/bin:$PATH" >> /etc/profile

#重新加载配置文件到内存，刷新修改信息，使修改生效
source /etc/profile
```



## 3. 配置

### 3.1 构建目录

```shell
#创建mysql用于存放数据的目录
mkdir -p /opt/mysql/data

#创建mysql用于保存日志的目录
mkdir -p /opt/mysql/logs
```



### 3.2 创建用户组和用户

```shell
#构建mysql用户组
groupadd mysql

#创建mysql用户并将该用户添加到刚刚创建好的用户组中
useradd -g mysql mysql
```



### 3.3 用户和用户组修改

```shell
#修改mysql安装目录的用户权限
chown -R mysql:mysql /usr/local/mysql & chmod -R 755 /usr/local/mysql

#修改mysql数据及日志存储目录的用户权限
chown -R mysql:mysql /opt/mysql & chmod -R 755 /opt/mysql
```



### 3.4 配置文件

```shell
#1. 编辑mysql启动配置文件my.cnf（命令）
vim /etc/my.cnf


#2. 输入以下内容：
[client]
port=9606
socket=/tmp/mysql.sock

[mysqld]
skip_ssl
port=9606
user=mysql
socket=/tmp/mysql.sock
basedir=/usr/local/mysql
datadir=/opt/mysql/data
log-error=/opt/mysql/logs/error.log
pid-file =/opt/mysql/logs/mysql.pid
transaction_isolation = READ-COMMITTED
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
lower_case_table_names = 1

sql_mode="STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,NO_ZERO_DATE,NO_ZERO_IN_DATE,ERROR_FOR_DIVISION_BY_ZERO"
```



## 4. 初始化MySQL

### 4.1 初始化

```shell
# mysql8如果不安装libaio-devel就会启动失败，所以建议在启动之前，先安装好与其有关的依赖包
yum install libaio-devel.x86_64

# 初始化mysql，此时/opt/mysql/orgs/下就已经有了日志文件，且初始密码也在该目录下的日志文件中
/usr/local/mysql/bin/mysqld --initialize --user=mysql

# 启动mysql服务
/usr/local/mysql/support-files/mysql.server start
```



### 4.2 设置mysql开机启动

```shell
# 这一个操作你可以认为是一个创建快捷方式的操作，且/etc/init.d是一个“开放目录”，即在里面的东西，无论在文件系统的哪个位置，随时都可以访问到
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld 

# 为“快捷方式”文件授权，同时设置mysql开机自启动
chmod 755 /etc/init.d/mysqld & chkconfig --add mysqld & chkconfig --level 345 mysqld on
```



### 4.3重启mysql服务

```shell
service mysqld restart
```





## 5. 账户

### 5.1 登录mysql

```shell
#1 通过shell命令打印mysql日志并过滤出其初始密码（$NF表示获取最后一列内容）
cat /opt/mysql/logs/error.log | grep 'password' | awk -F ' ' '{print $NF}'

#2 根据输出的字符串（初始密码登录mysql）登录mysql
mysql -uroot -p
```




### 5.2 修改密码及ip访问权限

```mysql
-- 1 （进入mysql后）修改root用户的密码
ALTER user 'root'@'localhost' IDENTIFIED BY '你的密码';

-- 2 （进入mysql后）为root用户创建映射，同时，开启允许全部ip访问root用户
CREATE USER 'root'@'%' IDENTIFIED WITH mysql_native_password  BY '你的密码';
GRANT ALL ON *.* TO 'root'@'%';

-- 3. 刷新修改信息
FLUSH PRIVILEGES;
```


至此 基于Mysql 8.0.18安装完成



## 附：

### 1. 关于端口开放的问题

	有时候我们需要开发mysql的端口，供外部通过该端口对mysql的资源进行访问，所以这个时候，我们必须要开放防火墙对通信的阻碍，以前我们在学校时可能习惯了使用“systemctl disable firewalld.service”，这种方式虽然可行，但是在实际的开发方式中为了安全考虑，不可能采用这种方式。
所以，这里以当前安装的mysql为例，讲一下怎么开放firewall端口的问题：

#### 1.0.在展开前，再次说明下，本文操作基于CentOS 7.2，CentOS7之前采用的是iptables，不是firewalld


#### 1.1.临时关闭防火墙（下次重启linux时，防火墙还是会打开）

   ```shell
   systemctl stop firewalld.service
   ```


#### 1.2.永久关闭防火墙（下次开启linux时，防火墙不会打开）

   ```shell
   systemctl disable firewalld.service
   ```


#### 1.3.开启防火墙（开机自动开启firewalld）

   ```shell
   systemctl enable firewalld
   ```


#### 1.4.永久开启一个端口（首先，确保你的防火墙启动了，可以用systemctl status firewalld检查一下，如果是running，就表示正常）

   ```shell
   firewall-cmd --permanent --zone=public --add-port=9606/tcp   #返回success就表示成功
   firewall-cmd --reload    #重载防火墙设置，上面的防火墙配置项才会生效
   ```

   



### 2.（待补充）关于不同业务场景下MySQL调优的问题