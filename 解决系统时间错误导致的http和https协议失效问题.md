## 解决系统时间错误导致的http和https协议失效问题

```shell
#yum安装ntp
yum install ntpdate

#更新同步系统时间
ntpdate cn.pool.ntp.org
```

