## 在cicd框架中新增两个test环境（test1、test2）

### 项目打包环境配置

配置两个子域名http://oms.api.test1.hxcapital.cn、http://oms.api.test2.hxcapital.cn指向test服务器同个ip的不同端口（9501、9502）

##### 前端

新增两个项目配置文件.env.test1、.env.test2

新增两个打包环境脚本npm run build:test1、npm run build:test2，当执行时分别去取.env.test1、.env.test2两个配置文件

**后端**

在各个微服务中的resource中新增两个项目配置文件application-test1.yml、application-test2.yml

将服务的port访问配置放到application-test1.yml、application-test2.yml中，并保证与application-test.yml的访问port不同

### git、github

新增两条容器分支（test1、test2）

### harbor

新增两个test环境镜像仓库目录（docker-harbor-test1、docker-harbor-test2）

### k8s、docker

在项目中根目录新增两个k8s应用部署配置文件(oms-test1.yml、oms-test2.yml)文件

​	deployment的元属性metadata下的名称（==name==）改为新的k8s应用名称oms-web-test1-deployment，命名空间（==namespace==）改为test1

​    app字段（pod）为新的test k8s容器名（oms-web-test1-pod） 

​    ==imagePullSecrets==下的name为新的harbor容器仓库目录（docker-harbor-test1） 

​    docker容器名称（containers下的name字段）改为新的test名（oms-web-test），避免容器名冲突

​    docker容器所使用的镜像（containers下的image字段）改为新的镜像名称（harborserver:9103/huaixin/oms-web-test1:latest），避免镜像名冲突 ==这里镜像名必须和远程镜像仓库（harbor）的存储路径对应==

#### jenkins

##### pieline

增加两个active选项分别为test1、test2对应项目打包配置文件的.env.test1、.env.test2和k8s部署配置文件oms-test1.yml、oms-test2.yml

