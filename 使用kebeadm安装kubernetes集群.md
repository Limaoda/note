## 使用kebeadm安装kubernetes集群

#### 1、kubernetes部署环境准备

* 两台2G或2G以上、2核及2核以上的计算机
* 所有计算机之间网络能互相连接（内网或公网）
* 关闭防火墙

```shell
# 关闭selinux（linux安全机制）
sed -i 's/enforcing/disabled/' /etc/selinux/config # 永久
setenforce 0 # 临时

# 关闭swap（k8s禁止虚拟内存以提高性能）
swapoff -a # 临时
sed -ri 's/.*swap.*/#&/' /etc/fstab # 永久

# 在master添加hosts(可选)
cat >> /etc/hosts << EOF
10.254.8.4 k8smaster
10.254.8.5 k8snode
EOF

# 设置网桥参数
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system

# 时间同步
yum install ntpdate -y
ntpdate time.windows.com
```

