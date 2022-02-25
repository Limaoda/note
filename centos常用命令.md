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

# 查看端口占用
netstat -lnp|grep 9200

# 查看进程
ps -ef|grep xxxx

# 杀死进程
kill -9 PID

# 给用户访问某个文件夹赋权
chown -R es:es /opt/ # 赋权es用户访问/opt文件夹

# 查看运行内存占用
ps -e -o 'pid,comm,args,pcpu,rsz,vsz,stime,user,uid' | grep oracle | sort -nrk5

# 查看内存信息
# total #总计物理内存的大小
# used #已使用多大
# free #可用有多少
# shared #多个进程共享的内存总额
# buff/cached #磁盘缓存的大小
# available #可用内存数
free -m

cat /proc/meminfo #查看RAM使用情况

top #实时显示进程的动态，查看进程占用内存的使用情况

# 查看磁盘空间使用情况
df -h

# 查看当前目录
ll -h


```

