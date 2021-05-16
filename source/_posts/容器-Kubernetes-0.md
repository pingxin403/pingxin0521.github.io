---
title: K8s 介绍
date: 2019-09-04 08:18:59
tags:
 - 容器
 - K8s
categories:
 - 容器
 - K8s
---

就在Docker容器技术被炒得热火朝天之时，大家发现，如果想要将Docker应用于具体的业务实现，是存在困难的——编排、管理和调度等各个方面，都不容易。于是，人们迫切需要一套管理系统，对Docker及容器进行更高级更灵活的管理。

<!--more-->

就在这个时候，K8S出现了。

**K8S，就是基于容器的集群管理平台，它的全称，是kubernetes。**

![Kg9KoT.png](https://s2.ax1x.com/2019/10/28/Kg9KoT.png)

Kubernetes这个单词来自于希腊语，含义是舵手或领航员。K8S是它的缩写，用“8”字替代了“ubernete”这8个字符。

和Docker不同，K8S的创造者，是众人皆知的行业巨头——**Google**。

然而，K8S并不是一件全新的发明。它的前身，是Google自己捣鼓了十多年的**Borg系统**。

K8S是2014年6月由Google公司正式公布出来并宣布开源的。

同年7月，微软、Red Hat、IBM、Docker、CoreOS、Mesosphere和Saltstack等公司，相继加入K8S。

之后的一年内，VMware、HP、Intel等公司，也陆续加入。

2015年7月，Google正式加入OpenStack基金会。与此同时，Kuberentes v1.0正式发布。

目前，kubernetes的版本已经发展到V1.13。

#### 优点

Kubernetes（k8s）是自动化容器操作的开源平台，这些操作包括部署，调度和节点集群间扩展。如果你曾经用过Docker容器技术部署容器，那么可以将Docker看成Kubernetes内部使用的低级别组件。Kubernetes不仅仅支持Docker，还支持Rocket，这是另一种容器技术。

使用Kubernetes可以：

- 自动化容器的部署和复制
- 随时扩展或收缩容器规模
- 将容器组织成组，并且提供容器间的负载均衡
- 很容易地升级应用程序容器的新版本
- 解决Docker跨机器容器之间的通讯问题
- 提供容器弹性，如果容器失效就替换它，等等...

实际上，使用Kubernetes只需一个[部署文件](https://github.com/kubernetes/kubernetes/blob/master/examples/guestbook/all-in-one/guestbook-all-in-one.yaml)，使用一条命令就可以部署多层容器（前端，后台等）的完整集群：

```
$ kubectl create -f single-config-file.yaml
```

kubectl是和Kubernetes API交互的命令行程序。

#### 架构

K8S的架构，略微有一点复杂，我们简单来看一下。

一个K8S系统，通常称为一个**K8S集群（Cluster）**。

这个集群主要包括两个部分：

- **一个Master节点（主节点）**
- **一群Node节点（计算节点）**

![Kg9Gl9.png](https://s2.ax1x.com/2019/10/28/Kg9Gl9.png)



一看就明白：Master节点主要还是负责管理和控制。Node节点是工作负载节点，里面是具体的容器。

深入来看这两种节点。

首先是**Master节点**

![Kg9wFO.png](https://s2.ax1x.com/2019/10/28/Kg9wFO.png)

Master节点包括API Server、Scheduler、Controller manager、etcd。

- API Server是整个系统的对外接口，供客户端和其它组件调用，相当于“营业厅”。
- Scheduler负责对集群内部的资源进行调度，相当于“调度室”。
- Controller manager负责管理控制器，相当于“大总管”。
- etcd:所有的持久性状态都保存在etcd中。Etcd同时支持watch，这样组件很容易得到系统状态的变化，从而快速响应和协调工作。

然后是**Node节点**。

![Kg9gmt.png](https://s2.ax1x.com/2019/10/28/Kg9gmt.png)

Node节点包括Docker、kubelet、kube-proxy、Fluentd、kube-dns（可选），还有就是**Pod**。

**Pod是Kubernetes最基本的操作单元。**一个Pod代表着集群中运行的一个进程，它内部封装了一个或多个紧密相关的容器。除了Pod之外，K8S还有一个Service的概念，一个Service可以看作一组提供相同服务的Pod的对外访问接口。

- Docker，不用说了，创建容器的。
- Kubelet，主要负责监视指派到它所在Node上的Pod，包括创建、修改、监控、删除等。
- Kube-proxy:Kube-proxy是一个简单的网络代理和负载均衡器。它具体实现Service模型，每个Service都会在所有的Kube-proxy节点上体现。根据Service的selector所覆盖的Pods, Kube-proxy会对这些Pods做负载均衡来服务于Service的访问者。
- Fluentd，主要负责日志收集、存储与查询。

![Kh2EOH.png](https://s2.ax1x.com/2019/10/30/Kh2EOH.png)

### 集群

集群是一组节点，这些节点可以是物理服务器或者虚拟机，之上安装了Kubernetes平台。下图展示这样的集群。注意该图为了强调核心概念有所简化。[这里](http://kubernetes.io/v1.1/docs/design/architecture.html)可以看到一个典型的Kubernetes架构图。

![Kg9L7V.png](https://s2.ax1x.com/2019/10/28/Kg9L7V.png)

上图可以看到如下组件，使用特别的图标表示Service和Label：

- Pod
- Container（容器）
- Label(label)（标签）
- Replication Controller（复制控制器）
- Service（enter image description here）（服务）
- Node（节点）
- Kubernetes Master（Kubernetes主节点）

**Pod**

Pod（上图绿色方框）安排在节点上，包含一组容器和卷。同一个Pod里的容器共享同一个网络命名空间，可以使用localhost互相通信。Pod是短暂的，不是持续性实体。你可能会有这些问题：

如果Pod是短暂的，那么我怎么才能持久化容器数据使其能够跨重启而存在呢？ 是的，Kubernetes支持卷的概念，因此可以使用持久化的卷类型。

是否手动创建Pod，如果想要创建同一个容器的多份拷贝，需要一个个分别创建出来么？可以手动创建单个Pod，但是也可以使用Replication Controller使用Pod模板创建出多份拷贝，下文会详细介绍。

如果Pod是短暂的，那么重启时IP地址可能会改变，那么怎么才能从前端容器正确可靠地指向后台容器呢？这时可以使用Service，下文会详细介绍。

**Lable**

正如图所示，一些Pod有Label（enter image description here）。一个Label是attach到Pod的一对键/值对，用来传递用户定义的属性。比如，你可能创建了一个"tier"和“app”标签，通过Label（tier=frontend, app=myapp）来标记前端Pod容器，使用Label（tier=backend, app=myapp）标记后台Pod。然后可以使用Selectors选择带有特定Label的Pod，并且将Service或者Replication Controller应用到上面。

**Replication Controller**

是否手动创建Pod，如果想要创建同一个容器的多份拷贝，需要一个个分别创建出来么，能否将Pods划到逻辑组里？

Replication Controller确保任意时间都有指定数量的Pod“副本”在运行。如果为某个Pod创建了Replication Controller并且指定3个副本，它会创建3个Pod，并且持续监控它们。如果某个Pod不响应，那么Replication Controller会替换它，保持总数为3.如下面的动画所示：

![KgCfD1.png](https://s2.ax1x.com/2019/10/28/KgCfD1.png)

如果之前不响应的Pod恢复了，现在就有4个Pod了，那么Replication Controller会将其中一个终止保持总数为3。如果在运行中将副本总数改为5，Replication Controller会立刻启动2个新Pod，保证总数为5。还可以按照这样的方式缩小Pod，这个特性在执行滚动升级时很有用。

当创建Replication Controller时，需要指定两个东西：

1. [Pod模板](http://kubernetes.io/v1.1/docs/user-guide/replication-controller.html#pod-template)：用来创建Pod副本的模板
2. [Label](http://kubernetes.io/v1.1/docs/user-guide/replication-controller.html#labels)：Replication Controller需要监控的Pod的标签。

现在已经创建了Pod的一些副本，那么在这些副本上如何均衡负载呢？我们需要的是Service。

**Service**

*如果Pods是短暂的，那么重启时IP地址可能会改变，怎么才能从前端容器正确可靠地指向后台容器呢？*

[Service](http://kubernetes.io/v1.1/docs/user-guide/services.html)是定义一系列Pod以及访问这些Pod的策略的一层**抽象**。Service通过Label找到Pod组。因为Service是抽象的，所以在图表里通常看不到它们的存在，这也就让这一概念更难以理解。

现在，假定有2个后台Pod，并且定义后台Service的名称为‘backend-service’，lable选择器为（tier=backend, app=myapp）。backend-service 的Service会完成如下两件重要的事情：

- 会为Service创建一个本地集群的DNS入口，因此前端Pod只需要DNS查找主机名为 ‘backend-service’，就能够解析出前端应用程序可用的IP地址。
- 现在前端已经得到了后台服务的IP地址，但是它应该访问2个后台Pod的哪一个呢？Service在这2个后台Pod之间提供透明的负载均衡，会将请求分发给其中的任意一个（如下面的动画所示）。通过每个Node上运行的代理（kube-proxy）完成。[这里](http://kubernetes.io/v1.1/docs/user-guide/services.html#virtual-ips-and-service-proxies)有更多技术细节。

下述动画展示了Service的功能。注意该图作了很多简化。如果不进入网络配置，那么达到透明的负载均衡目标所涉及的底层网络和路由相对先进。如果有兴趣，[这里](http://www.dasblinkenlichten.com/kubernetes-101-networking/)有更深入的介绍。

![KgCH8e.png](https://s2.ax1x.com/2019/10/28/KgCH8e.png)

有一个特别类型的Kubernetes Service，称为'[LoadBalancer](http://kubernetes.io/v1.1/docs/user-guide/services.html#type-loadbalancer)'，作为外部负载均衡器使用，在一定数量的Pod之间均衡流量。比如，对于负载均衡Web流量很有用。

**Node**

节点（上图橘色方框）是物理或者虚拟机器，作为Kubernetes worker，通常称为Minion。每个节点都运行如下Kubernetes关键组件：

- Kubelet：是主节点代理。
- Kube-proxy：Service使用其将链接路由到Pod，如上文所述。
- Docker或Rocket：Kubernetes使用的容器技术来创建容器。

**Kubernetes Master**

集群拥有一个Kubernetes Master（紫色方框）。Kubernetes Master提供集群的独特视角，并且拥有一系列组件，比如Kubernetes API Server。API Server提供可以用来和集群交互的REST端点。master节点包括用来创建和复制Pod的Replication Controller。

#### 单机实验环境搭建

本人用的是Ubuntu18，以下以此为例。

从阿里云上购买一台新加坡ECS，使用抢占式实例，每小时大概就0.05元不到，非常适合学习。

![YmnBqg.png](https://s1.ax1x.com/2020/05/07/YmnBqg.png)

**docker**

这个不用说了吧，K8S 专门盘 container 的，肯定要这个。

**关防火墙**

```
sudo ufw disable
```

关防火墙这个一般还是需要的，看自己的需求吧。

**关闭系统 swap**

```
sudo swapoff -a
```

这个不关就等着报错吧

##### K8S 环境

**添加 K8s 安装密钥**

先执行

```
  sudo apt update &&  sudo apt install -y apt-transport-https curl
```

然后执行

```
  curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg |  sudo apt-key add -
```

**配置 K8S 源**

```
  sudo touch /etc/apt/sources.list.d/kubernetes.list
```

这一步主要是为了安装 K8S 相关组件配置下载源，因为网络的问题分为两种形式吧。
 **如果你 VPN 挂好了**

```
  sudo  echo  "deb http://apt.kubernetes.io/ kubernetes-xenial main"  >> /etc/apt/sources.list.d/kubernetes.list
```

如果你没挂好，就老实用国内的吧。
 这里我选择阿里云的

```
  sudo  echo  "deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main"  >> /etc/apt/sources.list.d/kubernetes.list
```

种类没选择好的话接下来安装那一步就等着连接超时吧

**安装 kubeadm 及 kubelet 等工具**

正常安装

```
  sudo  apt-get update
```

更新完安装

```
  sudo  apt-get  install -y kubelet kubeadm kubectl
```

保持版本取消自动更新，这里也可以省略

```
  sudo apt-mark hold kubelet kubeadm kubectl
```

##### kubeadm 初始化

**初始化操作**

我这里用了阿里云镜像，用了1.18.2 版本，其他版本自行选择吧

```
# sudo kubeadm init --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.18.2 --pod-network-cidr=10.240.0.0/16

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.43.195:6443 --token lkftyz.om5nzn2nki2x3ibk \
    --discovery-token-ca-cert-hash sha256:b6e621f9c1cc2f844cd1633e4d405dfc74f4f1dbf58d480867d434afdcde946f 
```

**配置非 root 的操作**

```
  mkdir -p $HOME/.kube
  sudo  cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo  chown  $(id -u):$(id -g)  $HOME/.kube/config
```

**coredns 问题解决**

这个时候可以查看 pod 状态了，K8S 已经启动了支持服务，其中有 coredns 服务可能一直处于 pending 状态，那么我们需要做如下操作

**执行**

```
  kubectl apply -f https://docs.projectcalico.org/v3.10/manifests/calico.yaml
```

这样我们会安装 calico 网络插件，其实 flannel 也可以的哈哈。。
 这次查看 pod 状态

```
  kubectl get pods --all-namespaces
```

##### 运行相关

简单说一点点

**Master 隔离解除**

```
  kubectl taint nodes --all node-role.kubernetes.io/master-
```

成功后输出

```
  node/symoon untainted
```

**加入工作节点**

这项工作是集群的，单机可以忽略
 执行上面 init 出现的 join 结果

```
  kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>

# kubeadm token list (可以获取<token>的值)
# kubeadm token create (可以创建<token>新的值)

# --discovery-token-ca-cert-hash (以下命令可以获取 --discovery-token-ca-cert-hash的值)
# openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

加成功的话 ready-node 就会变成两个了

#### 单机实验环境搭建 - microk8s

通过 [microk8s](https://microk8s.io/) 可以非常快速的搭建起一个 Kubernetes 单机环境，安装极其非常方便，通过 `snap` 命令一键安装：

```shell
snap install microk8s --classic --channel=1.14/stable

#启动 microk8s
microk8s.start
microk8s.status


#查看版本信息
snap alias microk8s.kubectl kubectl

kubectl version

#安装插件
# 启用插件
microk8s.enable dns dashboard ingress

# 查看进度
kubectl get pods --all-namespaces

# 查看详情
kubectl describe pod --all-namespaces


#使用代理访问microk8s
# 找到pod名
kubectl get pods --all-namespaces | grep dashboard

# 查看pod的开放端口
kubectl describe --namespace kube-system pod/kubernetes-dashboard-6fd7f9c494-dgxlj

# 将pod的开放端口映射到本地
kubectl port-forward --namespace=kube-system --address=0.0.0.0 pod/kubernetes-dashboard-6fd7f9c494-dgxlj 8443:8443

#打开防火墙
iptables -I INPUT -p tcp --dport 8443 -j ACCEPT

iptables -t filter -L INPUT --line-numbers

```

浏览器打开 https://{Ubuntu_IP_address}:8443即可

使用token登录管理页面，下面是查看token的命令

```bash
# kubectl get secrets --all-namespaces | grep dashboard-token
# kubectl describe --namespace kube-system secrets kubernetes-dashboard-token-bhpxc
```

**使用代理**

上面直接使用端口转发的话，其实是绕过了service这一层，所以无法提供负载均衡的功能，只能固定访问一个pod。如果想使用使用service，可以设置代理

```bash
# 后台执行
kubectl proxy --accept-hosts=.* --address=0.0.0.0 &
```

浏览器访问
` http://{Ubuntu_IP_address}:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/`

> ubuntu 自带 snap 命令，可以直接执行，[centos 需要先安装 snap](https://www.jianshu.com/p/cd2c8d941e01)。

##### 添加一个nginx应用

![YmJAld.png](https://s1.ax1x.com/2020/05/07/YmJAld.png)

浏览器打开 https://{Ubuntu_IP_address}即可

#### 集群配置

Master节点上运行如下服务:

- etcd (etcd服务也可以单独运行，不一定要运行在Master节点上)
- kube-apiserver
- kube-controller-manager
- kube-scheduler
- Kubelet
- kube-proxy

Slave节点上运行如下服务：

- Kubelet
- kube-proxy

**环境介绍**

操作系统版本：centos linux 7.2 64bit

- Master节点：192.168.2.99/24 vm1
- Slave节点1：192.168.2.47/24 vm2

Kubernets和etcd软件采用yum方式安装
Kubernets版本：1.5.2
Etcd版本：2.2.5

**Master节点配置**

1. 通用配置文件

   ```shell
   # grep -v '^#'  /etc/kubernetes/config 
   KUBE_LOG_LEVEL="--v=0"
   KUBE_ALLOW_PRIV="--allow-privileged=false"
   KUBE_MASTER="--master=http://192.168.2.99:8080"  //apiserver的地址
   ```

2. apiserver配置文件

   ```shell
   # grep -v '^#'  /etc/kubernetes/apiserver 
   KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0" //apiserver的监听地址
   # KUBE_API_PORT="--port=8080"  //apiserver的监听端口
   # KUBELET_PORT="--kubelet-port=10250" //kubelet服务的监控端口
   KUBE_ETCD_SERVERS="--etcd-servers=http://0.0.0.0:10379"  //etcd服务的地址
   KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"  //cluster服务的网段
   KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota"  //这里为了避免做用户认证，取消掉了ServiceAccount参数
   KUBE_API_ARGS=""
   ```

3. controller-manager配置文件，无需特殊的配置

   ```
   # cat /etc/kubernetes/controller-manager 
   ```

4. etcd配置文件

   ```shell
   # grep -v '^#' /etc/etcd/etcd.conf 
   ETCD_NAME=default 
   ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
   ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:10379"  //监听地址和端口
   ETCD_ADVERTISE_CLIENT_URLS="http://0.0.0.0:10379"  //集群服务监听地址和端口
   ```

5. 通用配置文件

   ```shell
   # grep -v '^#' /etc/kubernetes/config 
   KUBE_LOGTOSTDERR="--logtostderr=true"
   KUBE_LOG_LEVEL="--v=0"
   KUBE_ALLOW_PRIV="--allow-privileged=false"
   KUBE_MASTER="--master=http://192.168.2.99:8080" //master服务的apiserver地址和端口
   ```

6. kubelet服务配置文件

   ```
   # grep -v '^#' /etc/kubernetes/kubelet 
   KUBELET_ADDRESS="--address=192.168.115.7" //slave的监听地址
   KUBELET_HOSTNAME="--hostname-override=192.168.115.7" //slave的主机名
   KUBELET_API_SERVER="--api-servers=http://192.168.115.5:8080"   //master服务的apiserver地址和端口
   KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=gcr.io/google_containers/pause"  //kubenet服务的启动需要依赖的镜像pull地址
   KUBELET_ARGS=""
   ```

7. kube-proxy的配置文件，一般无需特别的配置

   ```
   # cat /etc/kubernetes/proxy 
   ```

**Slave节点配置**

```
# yum -y install kubernetes 
```

1. 通用配置文件

   ```shell
   # grep -v '^#' /etc/kubernetes/config 
   KUBE_LOGTOSTDERR="--logtostderr=true"
   KUBE_LOG_LEVEL="--v=0"
   KUBE_ALLOW_PRIV="--allow-privileged=false"
   KUBE_MASTER="--master=http://192.168.2.99:8080" //master服务的apiserver地址和端口
   ```

2. kubelet服务配置文件

   ```
   # grep -v '^#' /etc/kubernetes/kubelet 
   KUBELET_ADDRESS="--address=192.168.115.7" //slave的监听地址
   KUBELET_HOSTNAME="--hostname-override=192.168.115.7" //slave的主机名
   KUBELET_API_SERVER="--api-servers=http://192.168.2.99:8080"   //master服务的apiserver地址和端口
   KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=gcr.io/google_containers/pause"  //kubenet服务的启动需要依赖的镜像pull地址
   KUBELET_ARGS=""
   ```

3. kube-proxy的配置文件，一般无需特别的配置

   ```
   # cat /etc/kubernetes/proxy 
   ```

**服务启动**

1. master节点

```
# systemctl start etcd
# systemctl start kube-apiserver
# systemctl start kube-controller-manager
# systemctl start kube-scheduler
# systemctl start kubelet
# systemctl start kube-proxy
```

2. slave节点

```
# systemctl start docker
# systemctl start kubelet
# systemctl start kube-proxy
```

**验证集群是否正常工作**

```
# kubectl get nodes //在master节点运行
# kubectl cluster-info
```

**导入pause镜像**

此镜像为k8s的根容器镜像，需要科学上网方式下载，因而下载完成后采用本地导入的方式
Pause.tar 百度网盘下载地址
链接: https://pan.baidu.com/s/1gireY350jPUmbQ80np9iGw 提取码: d9cm 

```
# docker load < pause.tar 
```



**注意事项**

在kubernetes系统中，对容器的要求是，需要一直在前台执行。



### 参考

1. <https://my.oschina.net/jamesview/blog/2994112>
2. <http://www.dockone.io/article/932>
3. <http://docs.kubernetes.org.cn/227.html>
4. <https://blog.51cto.com/ylw6006/2108447>
5. https://www.kancloud.cn/huyipow/kubernetes