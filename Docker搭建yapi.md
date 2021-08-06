## Docker搭建Yapi

##### 1、运行 MongoDB

```shell
# 创建存储卷
docker volume create mongo-data

#启动 MongoDB
docker run -d --name mongo-yapi -v mongo-data:/data/db -e MONGO_INITDB_ROOT_USERNAME=${USERNAME} -e MONGO_INITDB_ROOT_PASSWORD=${PASSWORD} mongo
```

##### 2、获取YAPI镜像

```shell
docker pull registry.cn-hangzhou.aliyuncs.com/anoyi/yapi
```

##### 3、创建自定义配置文件config.json

```shell
#创建文件
touch config.json

#编辑修改文件
vim config.json
```

```json
/* config.json */
{
  "port": "3000",
  "adminAccount": "534664357@qq.com",
  "timeout":120000,
  "db": {
    "servername": "mongo",
    "DATABASE": "yapi",
    "port": 27017,
    "user": "yourname",
    "pass": "yourpass",
    "authSource": "admin"
  }
}
```

##### 4、初始化 YAPI 数据库索引及管理员账号

```shell
docker run -it --rm --link mongo-yapi:mongo --entrypoint npm --workdir /yapi/vendors -v $PWD/config.json:/yapi/config.json registry.cn-hangzhou.aliyuncs.com/anoyi/yapi run install-server

#初始化管理员账号成功,账号名："534664357@qq.com"，密码："ymfe.org"
```

##### 5、启动Yapi服务

```shell
docker run -d --name yapi --link mongo-yapi:mongo --workdir /yapi/vendors -p 3000:3000 -v $PWD/config.json:/yapi/config.json registry.cn-hangzhou.aliyuncs.com/anoyi/yapi server/app.js
```

##### 6、访问YAPI服务

- 访问： [http://localhost:3000](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A3000)
- 登录账号：`534664357@qq.com`
- 密码：`ymfe.org`



**到第六步为止，YAPI本地Docker部署已经完成，后面为一些可选操作**

##### 7、手动构建YAPI镜像

```shell
# .dockerfile文件
FROM node:12.13.1 as builder
WORKDIR /yapi
RUN apk add --no-cache wget python make
ENV VERSION=1.9.2
RUN wget https://github.com/YMFE/yapi/archive/v1.9.2.zip
RUN unzip v1.9.2.zip && mv yapi-1.9.2 vendors
RUN cd /yapi/vendors && cp config_example.json ../config.json && npm install --production --registry https://registry.npm.taobao.org

FROM node:12.13.1
MAINTAINER 534664357@qq.com
WORKDIR /yapi/vendors
COPY --from=builder /yapi/vendors /yapi/vendors
EXPOSE 3000
ENTRYPOINT ["node"]
```

```shell
#使用.dockerfile文件构建本地镜像
docker build -f .dockerfile -t yapi .
```





##### 8、YAPI升级

```shell
# 1、停止并删除旧版容器
docker rm -f yapi

# 2、获取最新镜像
docker pull registry.cn-hangzhou.aliyuncs.com/anoyi/yapi

# 3、启动新容器
docker run -d --name yapi --link mongo-yapi:mongo --workdir /yapi -p 3000:3000 -v $PWD/config.json:/config.json registry.cn-hangzhou.aliyuncs.com/anoyi/yapi server/app.js
```

