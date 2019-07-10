# K8S

![](media/15510846090207/15543762927704.jpg)


## Docker


### 名词解读

**NameSpace**
隔离容器进程的视图，使其只能看到某些指定的内容。
> Linux 本身具有的一些 NameSpace：Mount, User, Network, UTS, IPC

**Cgroup**
限制容器进程的可用资源
/sys/fs/cgroup/*
> 例外：`/proc/` 如果没有特殊处理，在容器中使用 top 命令会查看到当前宿主机的信息。因为 proc 目录并不能感知到 Cgroup 对容器进程的限制

**Rootfs**
根文件系统

**Container Image**
容器镜像

**Image Registry**
镜像注册


### yum 安装 Docker CE


```bash
yum remove docker \
          docker-client \
          docker-client-latest \
          docker-common \
          docker-latest \
          docker-latest-logrotate \
          docker-logrotate \
          docker-selinux \
          docker-engine-selinux \
          docker-engine
                  
yum install -y yum-utils device-mapper-persistent-data lvm2
  
yum-config-manager --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
yum install docker-ce
systemctl start docker

# dockerd 配置文件路径 /etc/docker/[daemon.json|key.json]
{
    "data-root": "/home/docker"
}

```



### 源码安装

下载 rpm 包

```
https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-17.03.2.ce-1.el7.centos.x86_64.rpm
https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm
```

安装并启动
```bash
# 依赖包
yum install -y libcgroup policycoreutils-python libtool-ltdl
sudo rpm i docker-ce-*
systemctl start docker      
systemctl enable docker     # 开机自启动
systemctl status docker     # 查看当前该服务
systemctl list-unit-files   # 查看当前所有的开机自启动项
```

如果启动失败，尝试删除`rm -rf /var/run/docker*`文件后再重新启动
常用操作

`docker info/version` 查看当前 docker 的版本信息
```bash
Client:
 Version:      17.03.0-ce
 API version:  1.26
 Go version:   go1.7.5
 Git commit:   3a232c8
 Built:        Tue Feb 28 08:10:07 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.03.0-ce
 API version:  1.26 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   3a232c8
 Built:        Tue Feb 28 08:10:07 2017
 OS/Arch:      linux/amd64
 Experimental: false
```

```bash
docker pull hello-world     # 拉取 docker 镜像
docker image ls             # 列出当前可用的镜像
docker container ls         # 列出当前可用的容器
docker container list --all # 查看当前所有的容器列表
docker ps                   # 查看当前运行的容器
docker run hello-world      # 开始运行这个镜像
```

自己创建一个镜像


Now run the build command. This creates a Docker image, which we’re going to tag using -t so it has a friendly name.
`docker build -t friendlyhello .` 

Now let’s run the app in the background, in detached mode:
`docker run -d -p 4000:80 friendlyhello`

Now use docker container stop to end the process, using the CONTAINER ID, like so:
`docker container stop 1fa4ab2cf395`




### 优缺点

container 共享物理机的资源
virtual machine 是一个独立的操作系统


### 常规命令

**container**

```bash
docker build -t friendlyhello .  # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyhello  # Run "friendlyname" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyhello         # Same thing, but in detached mode
docker container ls                                # List all running containers
docker container ls -a             # List all containers, even those not running
docker container stop <hash>           # Gracefully stop the specified container
docker container kill <hash>         # Force shutdown of the specified container
docker container rm <hash>        # Remove specified container from this machine
docker container rm $(docker container ls -a -q)         # Remove all containers
docker image ls -a                             # List all images on this machine
docker image rm <image id>            # Remove specified image from this machine
docker image rm $(docker image ls -a -q)   # Remove all images from this machine
docker login             # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag  # Tag <image> for upload to registry
docker push username/repository:tag            # Upload tagged image to registry
docker run username/repository:tag                   # Run image from a registry
```

**stack**

```bash
docker stack ls                                            # List stacks or apps
docker stack deploy -c <composefile> <appname>  # Run the specified Compose file
docker service ls                 # List running services associated with an app
docker service ps <service>                  # List tasks associated with an app
docker inspect <task or container>                   # Inspect task or container
docker container ls -q                                      # List container IDs
docker stack rm <appname>                             # Tear down an application
docker swarm leave --force      # Take down a single node swarm from the manager
```


### 数据存储方式

[官方解释](https://docs.docker.com/storage/#good-use-cases-for-bind-mounts)

- `Volumes`: 由 Docker 负责创建与管理，非 Docker 进程无法修改
- `Bind mounts`: 主机上的文件，非 Docker 进程也能修改
- `tmpfs`: 只存储于主机内存中

### Namespace、Cgroups、rootfs

一个容器实际上是一个由 Linux Namespace、Linux Cgroups 和 rootfs 三个技术构建出来的进程的隔离环境。一个正在运行的 Linux 容器，可以被一分为二地看待：
1. 一组联合挂载在 /var/lib/docker/aufs/mnt 上的 rootfs，这一部分我们称为“容器镜像”（Container Image），是容器的静态视图。
2. 一个由 Namespace+Cgroups 构成的隔离环境，这一部分我们称为“容器运行时”（Container Runtime），是容器的动态视图。


### 常规操作

#### 制作建镜像文件

##### Dockerfile

**CMD、RUN、ENTRYPOINT 的区别**
`RUN` 是在当前的镜像文件上新增了一层 layout
`CMD` 可以理解为该镜像启动时默认的启动命令
`ENTRYPOINT` 是 CMD 语句的前置位置，其实完整的启动命令是 ENTRYPOINT CMD, 默认为`/bin/sh -c `

**WORKDIR**
意思是在这一句之后，Dockerfile 后面的操作都以这句指定的目录作为当前目录


##### Example

docker build -t hello_world .

#### 发布镜像文件

```bash
docker login
docker tag zyl-test:v0.0.1 hzzhangyile/zyl
docker push hzzhangyile/zyl
```




## Kubernetes

### 名词解释

| 类别	| 名称 |
| --- | --- |
| 资源对象	| Pod、ReplicaSet、ReplicationController、Deployment、StatefulSet、 DaemonSet、Job、CronJob、HorizontalPodAutoscaler |
| 配置对象	| Node、Namespace、Service、Secret、ConfigMap、Ingress、Label、 CustomResourceDefinition、 ServiceAccount |
| 存储对象	| Volume、Persistent Volume | 
| 策略对象	| SecurityContext、ResourceQuota、LimitRange |

**Pod**
K8S 可调度的最小单元

**CRI**
Container Runtime Interface

**CNI**
Container Network Interface

**PV & PVC & StorageClass**
Persistent Volume:  持久卷
Persistent Volume Claim: 持久卷声明
StorageClass:

### 设计思想
**从更宏观的角度，以统一的方式来定义任务之间的各种关系，并且为将来支持更多种类的关系留有余地。**

### 设计理念
**申明式 API 对应的“编排对象”和“服务对象”， 都是 K8S 项目中的 API 对象**

### 证书

| 组件 | 用到的证书 |
| --- | --- |
| etcd | 使用 ca.pem、kubernetes-key.pem、kubernetes.pem；| 
| kube-apiserver | 使用 ca.pem、kubernetes-key.pem、kubernetes.pem；| 
| kubelet | 使用 ca.pem；| 
| kube-proxy | 使用 ca.pem、kube-proxy-key.pem、kube-proxy.pem；| 
| kubectl | 使用 ca.pem、admin-key.pem、admin.pem；| 
| kube-controller-manager | 使用 ca-key.pem、ca.pem| 

### Install

![](media/15510846090207/15516697545765.jpg)


#### Install kubeadm

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF

# Set SELinux in permissive mode (effectively disabling it)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# Ensure `net.bridge.bridge-nf-call-iptables` is set to 1 in your sysctl config
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system

# Make sure that the br_netfilter module is loaded before this step. 
# This can be done by running `lsmod | grep br_netfilter`. 
# To load it explicitly call `modprobe br_netfilter`.

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable --now kubelet

swapoff -a  # 关闭 swap 

# install Container Network Interface (CNI) 
# For flannel to work correctly, you must pass --pod-network-cidr=10.244.0.0/16 to kubeadm init.
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml

# kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

kubeadm init

cp -i /etc/kubernetes/admin.conf $HOME/.kube/config


```

> [官网 CNI](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)

#### 安装 kubectl 自动补齐

```bash
yum install -y bash-completion
sh /usr/share/bash-completion/bash_completion 
kubectl completion bash > ~/.kube/completion.bash.inc
printf "
# Kubectl shell completion
source '$HOME/.kube/completion.bash.inc'
" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

#### 安装 helm 管理工具

```bash
wget https://storage.googleapis.com/kubernetes-helm/helm-v2.13.1-linux-amd64.tar.gz
tar -zxvf helm-v2.13.1-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/helm

cat <<EOF > rbac-config.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
EOF

kubectl apply -f rbac-config.yaml
helm init --service-account tiller --history-max 200

helm repo add gitlab https://charts.gitlab.io # 添加 gitlab repo
helm install --namespace default --name gitlab-runner # 安装 gitlab-runner  Helm Chart
```

#### 生成 token & token-ca-cert-hash
```
kubeadm token create
openssl x509 -in /etc/kubernetes/pki/ca.crt -noout -pubkey | openssl rsa -pubin -outform DER | sha256sum
```


---

### Components

#### Master Components
`etcd` Consistent and highly-available key value store used as K8S backing store for all cluster data.
`kube-apiserver` exposes the K8S API, it is the front-end for the K8S control plane.
`kube-scheduler` select a node for them to run on.
`cloud-controller-manager` runs controllers that interact with the underlying cloud providers
`kube-controller-manager` These controllers include:
* Node Controller: Responsible for noticing and responding when nodes go down.
* Replication Controller: Responsible for maintaining the correct number of pods for every replication controller object in the system.
* Endpoints Controller: Populates the Endpoints object (that is, joins Services & Pods).
- Service Account & Token Controllers: Create default accounts and API access tokens for new namespaces.  


#### Node Components
`kubelet` An agent, it make sure that containers are running in a pod.
`kube-proxy` maintain network rules
`Container Runtime`  The software that is responsible for running containers. eg: Docker, containerd, cri-o, rktlet and any implementation of the K8S CRI.

#### Addons

DNS

WebUI 

Container Resource Monitoring

Cluster-level Logging


---


### K8S Workloads

#### Pod

#### ReplicaSet 

#### Deployment


#### StatefulSet

`kubectl run -it --rm --restart=Never --image busybox:1.28 dns-test`
`nslookup ${Pod_Name}.${Headless_Service_Name}.${NameSpace}.svc.cluster.local`

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: zyl-nginx-service
  namespace: zyl
spec:
  clusterIP: None   # Headless Service
  ports:
  - port: 10080
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: zyl-nginx
  namespace: zyl
  labels:
    app: nginx
spec:
  serviceName: zyl-nginx-service
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-pv
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: nginx-pv
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: nfs
      resources:
        requests:
          storage: 1Gi
```

#### DaemonSet

#### Job

#### CronJob


### K8S Applications Management

#### Service
Publish Service
    LoadBalancer
    NodePort
    ClusterIP
    ExternalName
Nginx/HAproxy Service
External Load Balancer

`my-svc.my-namespace.svc.cluster.local`

This specification will create a new Service object named “my-service”,
which targets TCP port 9376 on any Pod with the "app=MyApp" label. 
```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```

#### Headless Service

#### ConfigMap

#### Secret

#### Ingress

#### External Load Banlancer


### ApiServer
功能:
* 整个集群管理的 API 接口： 所有对集群进行的查询和管理都是通过 API 进行的
* 集群内部各个模块之间通信的枢纽： 所有模块之间并不会互相调用，而是通过和API Server打交道完成这部分的工作
* 集群的安全控制:API Server提供的验证和授权保证了整个集群的安全


为了缓解各模块对API Server的访问压力，各功能模块都采用缓存机制来缓存数据，各功能模块定时从API Server获取指定的资源对象信息（LIST/WATCH方法），然后将信息保存到本地缓存，功能模块在某些情况下不直接访问API Server，而是通过访问缓存数据来间接访问API Server。

### 资源模型与资源管理

QoS Class:
Guaranteed  有保证的
Burstable   会上弹的
Besteffort  尽力




### 数据存储
persistent volume           持久卷
persistent volume claime    持久卷声明
storage class               存储件




   
   
### 命名空间 Namespace
   
   
| resource | privilege  |
| --- | --- |
| Persistent Volume | ❌ |
| StorageClass | ❌ | 
| ClusterRole | ❌ |
| ClusterRoleBinding | ❌ | 
| PersistentVolumeClaime | ✔️ |
| Role | ✔️ | 
| RoleBinding | ✔️ |
| ServiceAccount | ✔️ | 
| Service | ✔️ | 
| Deployment | ✔️ | 



## Service Mesh (Istio)

### Architecture

**Data plane**
The data plane is composed of a set of intelligent proxies (Envoy) deployed as sidecars. These proxies mediate and control all network communication between microservices along with Mixer, a general-purpose policy and telemetry hub.

**Control plane**
The control plane manages and configures the proxies to route traffic. Additionally, the control plane configures Mixers to enforce policies and collect telemetry.


**Envoy**
Envoy is deployed as a sidecar to the relevant service in the same Kubernetes pod.

**Mixer**
Mixer is a platform-independent component

**Pilot**
Pilot provides service discovery for the Envoy sidecars, traffic management capabilities for intelligent routing (e.g., A/B tests, canary rollouts, etc.), and resiliency (timeouts, retries, circuit breakers, etc.).

**Citadel**
Citadel enables strong service-to-service and end-user authentication with built-in identity and credential management.

**Galley**
Galley is Istio’s configuration validation, ingestion, processing and distribution component. 


### Rule Configuration

There are five traffic management configuration resources in Istio:   
`VirtualService`, `DestinationRule`, `ServiceEntry`, `Gateway`, and `Sidecar`:

A `VirtualService` defines the rules that control how requests for a service are routed within an Istio service mesh.

A `DestinationRule` configures the set of policies to be applied to a request after VirtualService routing has occurred. 

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  trafficPolicy:
    loadBalancer:
      simple: RANDOM
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

A `ServiceEntry` is commonly used to enable requests to services outside of an Istio service mesh.

A `Gateway` configures a load balancer operating at the edge of the mesh for HTTP/TCP ingress traffic to a mesh application or egress traffic to external services.

A `Sidecar` configures one or more sidecar proxies attached to application workloads running inside the mesh.
