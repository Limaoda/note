### 前置准备

- 至少 2 台 **2核4G** 的服务器
- CPU 必须为 x86 架构，暂时未适配 arm 架构的 CPU
- **CentOS 7.8**、 **CentOS 7.9** 或 **Ubuntu 20.04**

#### 安装docker

```bash
所有节点

curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
systemctl start docker
systemctl enable docker
docker version
20.10.8

systemctl status docker
```

#### docker镜像加速配置

```bash
mkdir /opt/data/docker -p
vim  /etc/docker/daemon.json

{
  "registry-mirrors": ["https://rnysp4y7.mirror.aliyuncs.com"],
  "insecure-registries":["harborserver:9103"],
  "graph":"/opt/data/docker",
  "log-driver":"json-file",
  "log-opts": {"max-size":"200m", "max-file":"3"}
}

service docker restart
```

#### 安装Kuboard-Spray

```bash
docker run -d \
  --restart=unless-stopped \
  --name=kuboard-spray \
  -p 80:80/tcp \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v ~/kuboard-spray-data:/data \
  eipwork/kuboard-spray:latest-amd64
  
    # 如果抓不到这个镜像，可以尝试一下这个备用地址：
  # swr.cn-east-2.myhuaweicloud.com/kuboard/kuboard-spray:latest-amd64
```

在浏览器打开地址 `http://这台机器的IP`，输入默认密码 `Kuboard123`，即可登录 Kuboard-Spray 界面。

