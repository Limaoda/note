## Docker

[官方文档](https://docs.docker.com/get-started/)

[官方镜像仓库](https://hub.docker.com/)

[国内镜像仓库](https://hub.daocloud.io/)

#### Docker安装

```shell
#清空docker旧版本相关
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
#安装yum相关工具链
sudo yum install -y yum-utils

#使用阿里国内源安装docker
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#安装docker引擎
sudo yum install docker-ce docker-ce-cli containerd.io

#启动docker
sudo systemctl start docker

#重新加载守护进程配置文件
systemctl daemon-reload

#重新启动docker
systemctl restart docker

#设置docker开机自启
systemctl enable docker
```



####  Docker常用命令（其它）

```shell
#使用镜像创建docker容器
docker run -d[后台运行] --name [容器名称] -p port:cport [name]

#递归参数,如强制递归删除所有容器[$()]
docker rm -f $(docker ps -aq)

#获取某个命令的帮助[--help]
docker run --help

#从容器内部拷贝文件或文件夹至宿主容器
docker cp [id]:/home/hello.js /home

#输出docker日志
docker logs [options]

#查看元数据
docker inspect [options]

#退出容器不保持容器运行状态
exit

#退出容器并保持容器运行状态
ctrl+p+q

#查看docker进程
docker top [options]

#查看docker版本
docker version

#修改docker镜像标签
docker tag [old_image] [name:tag]

```

#### Docker常用配置选项

```shell
-d 后台运行
-it 命令交互式
-p 端口映射
-P 随机映射端口
-v 数据卷映射
--name 容器命名
-e 环境配置
-c 运行shell脚本

```



#### Docker镜像命令

```shell
#查看docker镜像
docker images [options]

#删除docker镜像
docker rmi [id]

#拉取镜像
docker pull [name]
```

#### Docker容器命令

```shell
#查看容器信息
docker ps [options]

#以镜像运行容器 参数d为以后台方式运行 -it为以交互房方式运行 bashShell为控制台路径
docker run [options] [id|name] [bashShell] 

#启动容器
docker start [id]

#停止容器
docker stop [id]

#强制停止容器
docker kill [id]

#删除容器
docker rm [id]

#容器内外文件移动
docker cp directory [container]:directory #将宿主机某个目录下的所有内容复制到指定容器的某个目录下
docker cp [container]:directory directory #将指定容器的某个目录下所有内容复制到宿主机指定目录下

#进入当前正在运行的容器
docker exec [options] [id] [bashShell]

#进入当前正在运行的容器并执行
docker attach [id]
```

#### Docker可视化面板

```shell
#安装
docker run -d -p 6011:9000 \
--restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```

#### Docker容器数据卷

##### 1、用处：

容器只打包应用和依赖环境，容器内数据实时同步挂载到宿主机的映射位置上，将数据从容器内分离出去，避免受容器影响，**容器与宿主机**可以**同步数据**，**容器与容器间**也可以同步数据，使用了数据卷挂载的任何一端数据一旦被修改，另一端立即同步（即使容器被停止依旧生效）

##### 2、使用方式：

###### 命令

```shell
#将容器内部某个目录下所有数据同步到宿主机的某个目录下
docker run -it -v directory:directory [id] [bashShell]

#查看容器元数据中的mounts内容，得到数据卷挂载信息 rource为主机目录，Destination为容器目录
docker inspect [id]

#查看已挂载的数据卷信息
docker volume [options] ---- create | inspect |-ls | rm | prune 
```

###### 挂载方式

```shell
#匿名挂载 需指定容器内挂载目录,随机映射到宿主机docker的volume目录下 linux下fullPath为/var/lib/docker/volume
docker run --name nginx01 -d -P -v /etc/nginx nginx

#具名挂载 需指定卷名称和容器内挂载目录，映射位置同上
docker run --name nginx01 -d -v nginx-volume:/etc/nginx nginx

#指定路径挂载
docker run --name nginx01 -d -v -P /home/nginx:/etc/nginx nginx

#通过ro和rw改变操作权限，ro（readonly）下只能改变宿主机被挂载的目录内容，无法修改容器内部被挂载的目录内容，默认为rw（readwrite）双端可任意读写
docker run --name nginx01 -d -v nginx-volume:/etc/nginx:[ro|rw] nginx
```



![image-20210727142502332](C:\Users\HuQiaoDong\AppData\Roaming\Typora\typora-user-images\image-20210727142502332.png)

#### DockerFile

##### dockerfile基本使用

> ! dockerfile的基本指令全部是大写字母形式，且脚本按照从上至下顺序执行

```shell
#使用dockerfile构建镜像
docker build -f [dockerfile] -t name:tag . #-f指定dockerfile文件位置 -t指定镜像名称、标签和构建位置
```



##### 常用命令详解

```shell
#dockerfile
FROM 基础镜像 
MAINTAINER 作者信息
ENV 暴露环境变量
WORKDIR 容器工作目录
VOLUME 数据卷挂载
COPY 拷贝宿主文件至容器内
RUN 镜像构建阶段执行命令,可以有多个
EXPOSE 容器对外暴露的端口
CMD 使用镜像运行容器时作为默认命令执行,如果有多个CMD,则使用最后一个忽略其它
ENTRYPOINT 不同于CMD,这个命令会在容器启动时追加到手动指定的命令参数后,因此可以存在多个
```

#### Docker网络















