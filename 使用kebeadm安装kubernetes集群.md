## 安装kubernetes集群

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
192.168.79.132 k8smaster
192.168.79.133 k8snode
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

#### 2、安装Docker

> 见Docker和K8S集群文件

#### 3、安装kubeadm、kubelet、kubectl

```shell
# 添加k8s的阿里云yum源
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 使用yum安装kubeadm、kubelet、kubectl
yum install kubelet-1.19.4 kubeadm-1.19.4 kubectl-1.19.4 -y

# 执行
systemctl enable kubelet.service

#查看三个工具是否成功安装
yum list installed | grep kubelet
yum list installed | grep kubeadm
yum list installed | grep kubectl

# 查看安装的版本：
kubelet --version
```

* **Kubelet**：运行在cluster（集群）所有节点（由Master节点的API Server控制）上，负责**启动POD和容器**

* **Kubeadm**：用于**初始化**cluster（集群）的一个工具

* **Kubectl**：kubectl是**kubenetes命令行工具**，通过kubectl可以部署和管理应用，查看各种资源，创建，删除和更新组件；

#### 4、部署Kubernetes Master主节点

```shell
# 192.168.79.132为部署Master节点的机器ip地址
# 如果报错，有可能是环境准备阶段的配置没有立即生效，reboot重启主机再重新执行kubeadm init命令即可
echo 1 > /proc/sys/net/ipv4/ip_forward

kubeadm init --apiserver-advertise-address=192.168.79.132 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.19.4 --service-cidr=10.96.0.0/12 --pod-network-cidr=10.244.0.0/16

# kubeadm实际上做了一些脚本封装，包括拉取运行kebernetes所需要的一些镜像，安全证书配置等，当出现successful后，进行下一步
mkdir -p $HOME/.kube 

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl get nodes
```

#### 5、将node节点加入Kubernetes master节点

```shell
# 生成永久token
kubeadm token create --ttl 0
#jj3bvz.h8iwxt6uuhi1s4qp

# 查看ca证书sha256编码hash值
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
sha256:1d347c2e695d67a0db41538a62f3407d97d8c476bae4fcfb756349f406bf4981 xv

# 在node节点机器上执行，将node节点加入k8s集群中
kubeadm join 192.168.79.132:6443 --token jj3bvz.h8iwxt6uuhi1s4qp \
    --discovery-token-ca-cert-hash sha256:1d347c2e695d67a0db41538a62f3407d97d8c476bae4fcfb756349f406bf4981 --v=2

```



