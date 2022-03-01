master节点主机ip:192.168.0.101

worker节点主机ip:192.168.0.102

### 1、安装 k8s

```bash
# 所有节点

# 将docker镜像源地址临时暴露到系统全局
export REGISTRY_MIRROR=https://registry.cn-hangzhou.aliyuncs.com

# 安装kubelet
curl -sSL https://kuboard.cn/install-script/v1.21.x/install_kubelet.sh | sh -s 1.21.2

kubelet --version
# v1.21.2
```

### 2、安装docker

```bash
# 所有节点

# 使用第一步的阿里云镜像源安装docker
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

# 启动docker服务
systemctl start docker

# 设置docker开启自启
systemctl enable docker

# 查看docker版本
docker version
# 20.10.8

# 查看docker引擎工作状态
systemctl status docker
```

### 3、docker 配置

```bash
#加速镜像
#镜像存储位置  slave不需要

# 创建一个docker数据存储文件夹
mkdir /opt/data/docker -p
vim  /etc/docker/daemon.json

# max-size设置日志文件的最大容量，max-file设置文件句柄的最大打开个数，graph设置挂载数据的物理机目录路径
{
  "registry-mirrors": ["https://rnysp4y7.mirror.aliyuncs.com"],
  "insecure-registries":["harborserver:9103"],
  "graph":"/opt/data/docker",
  "log-driver":"json-file",
  "log-opts": {"max-size":"200m", "max-file":"3"}
}

# 重启docker使配置文件生效
systemctl restart docker

# 查看docker相关配置信息，验证刚刚的配置文件是否生效
docker info

# 查看硬盘使用情况
df -h
```

### 4、初始化 master 节点

在准备作为master节点的服务器上

```bash
# 查看网卡ip地址
ip address

# 全局暴露master主机的ip
export MASTER_IP=192.168.0.101

# Kubernetes 容器组所在的网段，该网段安装完成后，由 kubernetes 创建，事先并不存在于您的物理网络中，后续被Kubernetes的容器网络使用
export POD_SUBNET=10.100.0.0/16

# 替换 apiserver.demo 为 您想要的 dnsName
export APISERVER_NAME=k8smaster

# 将变量写入host文件
echo "${MASTER_IP}    ${APISERVER_NAME}" >> /etc/hosts

# 初始化master节点
curl -sSL https://kuboard.cn/install-script/v1.21.x/init_master.sh | sh -s 1.21.2 /coredns
```

### 5、设置master网络

```bash

# calico作为网络插件
# kubectl apply -f https://kuboard.cn/install-script/v1.21.x/calico-operator.yaml 

# 获取flannel网络插件配置文件
wget https://kuboard.cn/install-script/flannel/flannel-v0.14.0.yaml

# sed命令将flannel-v0.14.0.yaml文件中默认的k8s容器组网段10.244.0.0/16替换为10.100.0.0/16
sed -i "s#10.244.0.0/16#${POD_SUBNET}#" flannel-v0.14.0.yaml

# Kubernetes应用flannel
kubectl apply -f ./flannel-v0.14.0.yaml
```

### 6、查看master节点

```bash
# 等待readyed
kubectl get node

kubectl get pod -n kube-system
```

### 7、设置nodeport（k8s中的端口开放）

```bash
# https://kuboard.cn/install/install-node-port-range.html

# k8s端口组开放
vim /etc/kubernetes/manifests/kube-apiserver.yaml

###############################
#kube-apiserver.yaml
spec:
  containers:
  - command:
    - kube-apiserver
+   - --service-node-port-range=1-32726
###############################
```

### 8、重启apiserver

```bash
# 获得 apiserver 的 pod 名字
export apiserver_pods=$(kubectl get pods --selector=component=kube-apiserver -n kube-system --output=jsonpath={.items..metadata.name})

# 删除 apiserver 的 pod
kubectl delete pod $apiserver_pods -n kube-system
```

#### 验证结果

```bash
kubectl describe pod $apiserver_pods -n kube-system

###############################
spec:
  containers:
  - command:
    - kube-apiserver
    - --service-node-port-range=1-32726
###############################
# 此时为成功
```

### 9、将子节点加入集群

在master节点的服务器上

```bash
kubeadm token create --print-join-command
```

在准备作为worker节点的服务器上

```bash
# kubeadm token create 命令的输出
export MASTER_IP=192.168.0.101

# 将master主机信息写入子节点服务器host中
echo "192.168.0.23   k8smaster" >> /etc/hosts

# 子节点加入集群
kubeadm join apiserver.demo:6443 --token mpfjma.4vjjg8flqihor4vt     --discovery-token-ca-cert-hash sha256:6f7a8e40a810323672de5eee6f4d19aa2dbdb38411845a1bf5dd63485c43d303
 
# 等待readyed
kubectl get nodes
```


```bash
# 可选
systemctl daemon-reload
systemctl restart kubelet

# node noready的解决方案
https://blog.csdn.net/xiaohuixing16134/article/details/102784269
```

### 10、使用配置文件生成k8s应用

```bash
  mkdir -p /opt/k8s/service
  
  # 进入目录
  cd /opt/k8s/service
  
  vim tomcat-nginx.yaml
```

创建一个nginx应用和一个tomcat应用
```yaml
# tomcat-nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx:1.12.0-alpine
        ports:
        - containerPort: 80

---

apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: default
spec:
  selector:
    app: nginx-pod
  clusterIP: None
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deployment
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: tomcat-pod
  template:
    metadata:
      labels:
        app: tomcat-pod
    spec:
      containers:
      - name: tomcat
        image: tomcat:8.5-jre10-slim
        ports:
        - containerPort: 8080

---

apiVersion: v1
kind: Service
metadata:
  name: tomcat-service
  namespace: default
spec:
  selector:
    app: tomcat-pod
  clusterIP: None
  type: ClusterIP
  ports:
  - port: 8080
    targetPort: 8080
```

### 11、容器管理

```bash
# 创建命名空间
kubectl create ns default
kubectl apply -f tomcat-nginx.yaml
# 删除
kubectl delete deployment tomcat-deployment -n default
kubectl delete service tomcat-service  -n default

kubectl delete deployment nginx-deployment -n default
kubectl delete service nginx-service  -n default

# 查看
kubectl get pod -o wide -n default
kubectl get deploy -o wide -n default
kubectl get svc  -o wide  -n default

# 访问容器服务
curl [容器ip]:8080

# 进入pod容器
kubectl exec -it ${pod_name} -n default -- /bin/sh
```

### 11、ingress反向代理

```bash
mkdir -p /opt/k8s/ingress
  
cd /opt/k8s/ingress
  
# 获取两个用于ingress反向代理的配置文件
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/provider/baremetal/service-nodeport.yaml
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/mandatory.yaml

# 获取ingress镜像
docker pull suisrc/ingress-nginx:0.30.0
```

#### 修改配置文件

```yaml
# service-nodeport.yaml
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
+     nodePort: 80
-   - name: https
-     port: 443
-     targetPort: 443
-     protocol: TCP
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
```

```yaml
# mandatory.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---

kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: tcp-services
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: udp-services
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: nginx-ingress-clusterrole
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - endpoints
      - nodes
      - pods
      - secrets
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - "extensions"
      - "networking.k8s.io"
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - "extensions"
      - "networking.k8s.io"
    resources:
      - ingresses/status
    verbs:
      - update

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  name: nginx-ingress-role
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - pods
      - secrets
      - namespaces
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - configmaps
    resourceNames:
      # Defaults to "<election-id>-<ingress-class>"
      # Here: "<ingress-controller-leader>-<nginx>"
      # This has to be adapted if you change either parameter
      # when launching the nginx-ingress-controller.
      - "ingress-controller-leader-nginx"
    verbs:
      - get
      - update
  - apiGroups:
      - ""
    resources:
      - configmaps
    verbs:
      - create
  - apiGroups:
      - ""
    resources:
      - endpoints
    verbs:
      - get

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: nginx-ingress-role-nisa-binding
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: nginx-ingress-role
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: ingress-nginx

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: nginx-ingress-clusterrole-nisa-binding
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: nginx-ingress-clusterrole
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: ingress-nginx

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
- replicas: 1
+ replicas: 2
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
      annotations:
        prometheus.io/port: "10254"
        prometheus.io/scrape: "true"
    -spec:
      # wait up to five minutes for the drain of connections
      terminationGracePeriodSeconds: 300
      serviceAccountName: nginx-ingress-serviceaccount
      nodeSelector:
        kubernetes.io/os: linux
      containers:
        - name: nginx-ingress-controller
_         image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.30.0
+         image: suisrc/ingress-nginx:0.30.0
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
            - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
            - --publish-service=$(POD_NAMESPACE)/ingress-nginx
            - --annotations-prefix=nginx.ingress.kubernetes.io
          securityContext:
            allowPrivilegeEscalation: true
            capabilities:
              drop:
                - ALL
              add:
                - NET_BIND_SERVICE
            # www-data -> 101
            runAsUser: 101
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
            - name: https
              containerPort: 443
              protocol: TCP
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10
          lifecycle:
            preStop:
              exec:
                command:
                  - /wait-shutdown

---

apiVersion: v1
kind: LimitRange
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  limits:
  - min:
      memory: 90Mi
      cpu: 100m
    type: Container
```



```bash
kubectl apply -f mandatory.yaml
kubectl apply -f service-nodeport.yaml

# 查看
kubectl get all -n ingress-nginx
kubectl get pod -n ingress-nginx -o wide
kubectl get deploy -n ingress-nginx -o wide
kubectl get svc -n ingress-nginx -o wide

# 查看pod是否亲和
kubectl delete service ingress-nginx -n ingress-nginx
kubectl delete deploy  nginx-ingress-controller -n ingress-nginx
```



#### 配置反向代理

```bash
vim ingress-http-default.yaml
```

```yaml
# ingress-http-default.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-http
  namespace: default
spec:
  rules:
  - host: nginx.api.test.hxcapital.cn
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx-service
          servicePort: 80
  - host: tomcat.api.test.hxcapital.cn
    http:
      paths:
      - path: /
        backend:
          serviceName: tomcat-service
          servicePort: 8080
```

```bash
kubectl apply -f ingress-http-default.yaml
kubectl get ing ingress-http -n default
kubectl describe ing ingress-http  -n default
```

#### 配置dns解析

ingress所在工作节点ip

### 13、Kuborad

#### 参考

    https://kuboard.cn/install/v3/install-in-k8s.html#%E5%AE%89%E8%A3%85

#### 安装

    kubectl apply -f https://addons.kuboard.cn/kuboard/kuboard-v3.yaml

#### 测试

    watch kubectl get pods -n kuboard

#### 使用

```txt
http://kuboard.tech.hxcapital.cn:30080
输入初始用户名和密码，并登录
用户名： admin
密码： Kuboard123
```

### 14、常用命令

#### 其它

```bash
kubectl apply -f ${xxxx.yaml}

# kubectl命令帮助文档 -h参数
# 查看kubectl所有操作
kubectl -h/kubectl --help

# 查看create命令的所有option
kubectl create -h
....
```

#### 创建

```bash
# 参数
--image # 镜像
--replicas # 副本
# 创建一个命名空间（命名空间用于隔离资源[service、deployment、pod]）
kubectl create ns ${namespace} 

# 创建一个pod容器(拉取一个容器镜像并运行容器)
kubectl run ${pod_name} --image ${image_name}

# 创建一个deployment控制器并指定副本
kubectl run ${deploy_name} --image ${image_name} --
```

#### 删除

```bash
# 删除指定命名空间下的所有资源 
kubectl delete all --all -n ${namespace}

# 删除指定容器
kubectl delete pod ${pod_name}/${pod_id}

# 删除指定deployment
kubectl delete deploy ${deploy_name}/${deploy_id}

# 删除指定service
kubectl delete svc ${service_name}/${service_id}

# 删除
```

#### 查看

```bash
# 参数
-owide # 拓展信息
-A # 全部命名空间资源
-n # 指定命名空间
```



#### 修改

3389



### 搭建新环境

1、修改oms-test.yml的所有namespace为test1并另存为oms-test1.yml

2、在test控制节点的服务器上拉取ingress-nginx镜像，修改两个配置文件

3、配置反向代理，将service暴露出的访问端口使用域名做解析

4、将ingress配置文件和反向代理配置文件应用于k8s集群