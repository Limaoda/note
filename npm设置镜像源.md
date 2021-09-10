1、命令行临时使用指定镜像（淘宝）

```shell
npm --registry https://registry.npm.taobao.org install express
```

2、命令行永久更改使用指定镜像（淘宝）

```shell
npm config set registry https://registry.npm.taobao.org
```

3、安装cnpm替代npm

```shell
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

4、查看目前使用的镜像源

```shell
npm config get registry
```

5、获取缓存目录

```shell
npm config get cache
# 或
npm root -g
# mac下： /Users/apple/.npmrc  ("apple"是自己的mac用户名)
# window下：%APPDATA%/npm/node_modules
```

6、安装npm源管理工具

```shell
npm install -g nrm
```

7、查看依赖树

```shell
npm ls express
```

8、查看npm源上的所有版本安装包

```shell
npm view express versions # 以express为例
```

9、安装最新包

```shell
npm install express@next # 以express为例
```

