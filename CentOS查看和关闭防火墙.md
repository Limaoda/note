## CentOS7查看和关闭防火墙

```shell
# 查看防火墙状态
firewall-cmd --state

# 停止防火墙
systemctl stop firewalld.service

#禁止防火墙开机启动
systemctl disable firewalld.service 
```



