准备工作：

1.确保linux系统为Centos7.x版本

2.确保有一台可以使用vpn的机器，用于下载被墙的安装包

3.确保系统上有yum包管理工具

4.确保系统的1080端口放行

### 1.1 安装Shadowsocks客户端

#### 安装epel扩展源
采用Python包管理工pip安装。

```shell
sudo yum -y install epel-release
sudo yum -y install python-pip
```

#### 安装Shadowsocks客户端

```shell
sudo pip install shadowsocks
```

### 1.2 配置Shadowsocks客户端

#### 新建配置文件

```shell
sudo mkdir /etc/shadowsocks
sudo vi /etc/shadowsocks/shadowsocks.json
```

#### 添加配置信息

```txt
{
    "server":"1.1.1.1",
    "server_port":1035,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"password",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false,
    "workers": 1
}
```

>参数说明：
>server：Shadowsocks服务器地址
>server_port：Shadowsocks服务器端口
>local_address：本地IP
>local_port：本地端口
>password：Shadowsocks连接密码
>timeout：等待超时时间
>method：加密方式
>workers:工作线程数
>fast_open：true或false。开启fast_open以降低延迟，但要求Linux内核在3.7+。开启方法 `echo 3 > /proc/sys/net/ipv4/tcp_fastopen`

#### 配置自启动

① 新建启动脚本文件/etc/systemd/system/shadowsocks.service，内容如下：

```txt
[Unit]
Description=Shadowsocks

[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/sslocal -c /etc/shadowsocks/shadowsocks.json

[Install]
WantedBy=multi-user.target
```

 ② 启动Shadowsocks客户端

```shell
systemctl enable shadowsocks.service
systemctl start shadowsocks.service
systemctl status shadowsocks.service
```

③解决不支持aes-256-gcm加密协议问题

```shell
 pip install https://github.com/shadowsocks/shadowsocks/archive/master.zip -U
 
 systemctl start shadowsocks
 
 systemctl status shadowsocks
```



#### 验证Shadowsocks客户端是否正常运行

```shell 
curl --socks5 127.0.0.1:1080 http://httpbin.org/ip
```

正常运行的话返回

```txt
{
	"origin": "x.x.x.x" #你的Shadowsock服务器IP
}
```



### 1.3 安装配置Privoxy

> Shadowsocks是一个 socket5 服务，我们需要使用 Privoxy 把流量转到 http／https 上

#### 下载安装文件

```shell
# 这里一般情况下是无法通过wget拿到安装包的，需要使用一台有vpn的机器访问网站下载安装包后拷贝到centos所在机器上
wget http://www.privoxy.org/sf-download-mirror/Sources/3.0.26%20%28stable%29/privoxy-3.0.26-stable-src.tar.gz
tar -zxvf privoxy-3.0.26-stable-src.tar.gz
cd privoxy-3.0.26-stable
```

#### 安装并编译privoxy

```shell
yum install install autoconf automake libtool #安装编译工具
autoheader && autoconf ./configure make && make install

```

#### 配置privoxy

```shell
vi /usr/local/etc/privoxy/config
```
找到以下两句，确保没有注释掉
```txt
listen-address 127.0.0.1:8118   # 8118 是默认端口，不用改，下面会用到
forward-socks5t / 127.0.0.1:1080 . # 这里的端口写 shadowsocks 的本地端口（注意最后那个 . 不要漏了）
```

> vim编辑器使用:/加上需要搜索的字符串进行快速定位，例如：
>
> :/listen-address

#### 启动privoxy

```shell
privoxy --user privoxy /usr/local/etc/privoxy/config
```

### 1.4 配置 /etc/profile

#### 编辑profile文件

```shell
vi /etc/profile
```

 添加下面两句：

```txt
export http_proxy=http://127.0.0.1:8118       #这里的端口和上面 privoxy 中的保持一致
export https_proxy=http://127.0.0.1:8118
```

重新加载配置文件

```shell
source /etc/profile
```

### 1.5 测试

```shell
curl www.google.com
```

返回一大堆 HTML 则说明 shadowsocks 正常工作了。

### 1.6 使用注意事项

#### 关闭shadowsocks代理

```shell
vi /etc/profile
```
注释以下两行
```txt
# export http_proxy=http://127.0.0.1:8118       #这里的端口和上面 privoxy 中的保持一致
# export https_proxy=http://127.0.0.1:8118
```



