## 虚拟机NAT网络模式下设置固定ip

```shell
cd /etc/sysconfig/network-scripts

vi ifcfg-ens33 
```

```shell
# ifcfg-ens33文件

TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"  # 这里修改为static
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="f61698de-3ced-46f8-8a6c-b5af229d7615"
DEVICE="ens33"
IPADDR="192.168.79.136" # 主机的ip地址
NETWORK="255.255.255.0" # 子母掩码
ONBOOT="yes"
~

```



