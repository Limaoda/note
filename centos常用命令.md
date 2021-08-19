## centos7.x常用命令

```shell
# 查看发行版版本信息
cat /etc/redhat-release

# 查看主机名
hostname 

# 编辑主机名，使用这个命令会立即生效且重启也生效
hostnamectl set-hostname centos77.magedu.com

# 编辑hosts文件，给127.0.0.1添加hostname
vim /etc/hosts

# 获取机器默认网卡，通常是 eth0
ip route show

# 显示默认网卡的IP地址
ip address

#查看cpu信息
lscpu

# 查看进程
ps -ef|grep xxxx

# 杀死进程
kill -9 PID
```

