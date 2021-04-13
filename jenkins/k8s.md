## Kubernetes(k8s)

### 第一部分： k8s 概念和架构

第二部分：从零开始搭建k8s集群 1.基于客户端工具 kubeadm. 2. 基于二进制包方式

第三部分 k8s核心概念

1. Pod Controller Service Ingress 
2. RBAC Helm等

第四部分 搭建集群监控平台系统

第五部分 从零搭建高可用k8s集群

第六部分 在集群环境部署项目

### K8s 概述

K8s 是谷歌2014年开源的容器化集群管理系统。使用k8s 进行容器化应用部署。使用k8s 利于应用扩展，k8s 目标是实施让部署容器化应用更加简洁和高效

#### 2.k8s 特性

1. 自动装箱 ： 基于容器对应用运行环境

2. 自动修复(自我治愈)
3. 水平扩展
4. 服务发现 用户不需要使用额外的服务发现机制
5. 滚动更新：
6. 版本回退 可以根据应用部署情况
7. 密钥和配置管理
8. 存储编排
9. 批处理

Master(主控节点)和 node(工作节点)

Master组件

- Apiserver 集群统一入口，以restful方式提供，交给etcd存储
- Scheduler 节点调度，选择node节点应用部署
- controller-manager 处理集群中常规后台任务，一个资源对应一个控制器
- etcd 存储，用于各种数据，存储系统，用于保存集群相关的数据。

node组件

- Kubelet master派到node节点代表，管理本机容器
- Kube-proxy 提供网络代理，负载均衡等操作。

### 3.k8s概念

- Pod 是k8s中最小的部署单元，一组容器的集合 共享网络 生命周期是短暂的
- Controller 确保预期的pod 副本数量 无状态应用部署。有状态应用部署 确保所有的node 运行同一个pod 一次性任务和定时任务
- Service 定义一组pod的访问规则

目前生产部署k8s集群主要有两种方式：

1.kubeadm  是一个k8s 部署工具，提供kubeadm init 和 kubeadm join, 用于快速部署k8s集群。

2.二进制包 从GitHub 下载发行版的二进制包，手动部署每个组件



#### 如何快速编写yaml 文件

第一种，使用kubectl create 命令生成yaml文件

```bash
kubectl create deployment web --image=nginx -o yaml --dry-yaml >m1.yaml
```



第二种，使用kubectl get 命令导出yaml文件

```bash
kubectl get deploy nginx -o=yaml --export > m2.yaml
```

### Pod概述

pod是k8s 的系统中可以创建和管理的最小单元。一个pod可以包含一个或多个容器，因此它可以被看作是内部容器的逻辑宿主机。Pod的设计理念是为了支持多个容器在一个Pod中共享网络和文件系统。处在一个Pod中的多个容器共享一下资源：

- PID命名空间：Pod中不同的应用程序可以看到其他应用程序的进程ID。
- network命名空间：Pod中多个容器处在相同的网络命名空间，因此能够访问的IP和端口范围是相同的。也可以通过localhost访问。
- IPC命名空间：Pod中的多个容器共享Inner-process Communication 命名空间。
- Volumes: Pod中各个容器可以共享在Pod中定义分存储卷(Volume)。

Pod、容器、Node(工作主机)之间的关系如下图所示：

![image-20210305191531796](/Users/dengwenqi/Library/Application Support/typora-user-images/image-20210305191531796.png)

Pod是k8s的最重要概念，每一个Pod都有一个特殊的被称为"根容器"的Pause容器。Pause容器队友的镜像属于k8s平台的一部分，除了Pause容器，每个Pod 还包含一个或多个紧密相关的用户业务容器。

#### Pod的定义

通过yaml文件或者json描述pod和其内容器的运行环境和期望状态，例如一个最简单的运行nginx应用的pod，定义如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: nginx
	labels:
		app: nginx
spec:
	containers:
	- name: nginx
	  image: nginx
	  ports:
	  - containerPort: 80
```

> 在生产环境中，推荐使用Deployment,StatefulSet,Job 或者CronJob等控制器来创建Pod,而不是直接创建。

```bash
kubectl apply -f nginx-pod.yaml
```

简单分析Pod定义文件：

- metadata: 用于唯一区分对象的元数据，包括 name, UID 和namespace
- Label: 是一个个的key/value 对，定义这样的label到Pod后，其他控制器对象可以通过这样的label来定位到此Pod,从而对Pod进行管理。
- spec: 其他描述信息，包括Pod中运行的容器，容器中运行的应用等等。不同类型的休息拥有不同的spec定义。

> 如果希望从外部访问nginx，我们需要创建Service对象来暴露IP和port

#### Pod的生命周期

Pod的生命周期是Replication Controller 进行管理的。一个Pod的生命周期过程包括：

- 通过yaml 或json 对Pod进行描述
- Apiserver(运行在Master主机)收到创建Pod的请求后，将此Pod对象的定义存储在etcd中
- Scheduler(运行在Master主机) 将此pod分配到其他Node
- Pod内所有容器运行结束后此Pod也结束

在整个过程中Pod分为5个状态：

- Pending: Pod定义正确，提交到Master，但其所包含的容器镜像还未完全创建。通常Master对Pod进行调度需要一些时间，Node进行容器镜像下载也需要一些时间。

- Running: Pod已经被分配到某个Node上，并且所有的容器都被创建完毕，至少有一个容器正在运行中，或者有容器正在启动或重启中。

- Succeeded：Pod中所有的容器都成功运行结束，并且不会被重启。这是Pod的一种最终状态。

  Failed：Pod中所有的容器都运行结束了，其中至少有一个容器是非正常结束的（exit code不是0）。这也是Pod的一种最终状态。

  Unknown：无法获得Pod的状态，通常是由于无法和Pod所在的Node进行通信。

#### Restart policy

定义pod时，可以指定restartPolicy 字段，表明此Pod中的容器在任何条件下会重启。

- Always: 只要退出就重启
- OnFailure: 失败退出时(exit code不是0)才重启
- Never： 永远不重启

#### Pod镜像拉取策略

- IfNotPresent: 默认值， 镜像在宿主机上不存在时才拉取
- Always 每次创建Pod 都会重新拉取一次镜像
- Never: Pod 永远不会主动拉取这个镜像

#### Pod中资源限制

Pod和Container 的资源请求和限制

Spec.containers[].resources.limits.cpu

Spec.containers[].resources.limits.memory

Spec.container[].resources.requests.cpu

spec.containers[].resources.requests.memory

```yaml
kind: Pod
spec:
	containers:
	- name: db
		resources:
			# 调度
			requests:
				memory: "64Mi"
				cpu: "250m"
			# 最大
			limits:
				memory:"128Mi"
				cpu: "500m"
```

#### Pod的健康检查

容器检查+ 应用层面健康检查

livenessProbe(存活检查) 如果检查失败，将杀死容器，根据Pod的restartPolicy 来操作。

readinessProbe(就绪检查) 如果检查失败，k8s 会把 Pod 从service endpoints中剔除。

Probe支持以下三种检查方法：

- httpGet 发送HTTP请求，返回200-400范围状态码为成功。
- Exec 执行shell 命令返回状态码为0为成功
- tcpSocket 发起TCP socket 建立成功。

```yaml
spec:
	containers:
	-	name liveness
		livenessProbe:
			exec:
				command:
					- cat
					- /tmp/healthy
			initialDelaySeconds: 5
			periodSeconds: 5
```

#### Pod创建流程图

![image-20210306104356638](/Users/dengwenqi/Library/Application Support/typora-user-images/image-20210306104356638.png)

#### 影响调度的属性

1. Pod资源限制对Pod调度产生影响
2. 节点选择器标签影响Pod调度

```yaml
spec:
	nodeSelector:
		env_role: dev
```

![image-20210306105857545](/Users/dengwenqi/Library/Application Support/typora-user-images/image-20210306105857545.png)

```bash
# 通过节点选择器选择标签
kubectl label node env_role=prod
```

节点亲和性 nodeAffinity 和之前nodeSelector 基本一样的，根据节点上的标签约束来绝对Pod调度，到哪些节点上。 

- 硬亲和性 约束条件必须满足 
- 软亲和性

```yaml
spec:
	affinity:
		nodeAffinity:
			requestdDuringSchedulingIngnoreDuringExecution:
				nodeSelectorTerms:
				- matchExpressions:
					- key: env_role
						operator: In
						values:
						- dev
						- test
```

影响Pod调度

- 污点和污点容忍

- nodeSelector和nodeAffinity: Pod 调度到某些节点上，调度时候实现

- Taint 污点：节点不做普通分配调度，是节点属性

  - 专用节点
  - 配置特定硬件节点
  - 基于Taint驱逐 

  查看节点污点情况

  ```bash
  kubectl describe node k8smaster | grep Taint
  kubectl create deployment web --image=nginx
  kubectl get pods -o wide 
  kubectl scale deployment web --replicas=5
  kubectl delete deployment web
  kubectl taint node k8snode1 env_role=yes:NoSchedule
  kubectl describe node k8snode1 | grep Taint
  ```

  污点值有三个：

  - NoSchedule 一定不被调度
  - PreferNoSchule 尽量不被调度
  - NoEcecute 不会调度，并且还会驱逐Node已有Pod

  为节点添加污点

  ```bash
  kubectl taint node [node] key=value:污点三个值
  ```

  ### controller

  #### 什么是controller

  在集群上管理和运行容器的对象

  #### Pod和Controller关系

  - Pod是通过controller 实现应用的运维
  - 比如伸缩，滚动升级等等
  - Pod 和controller 通过label 标签建立关系

- ``



```bash
kubectl set image deployment web nginx:1.15
```

![image-20210306225012886](/Users/dengwenqi/Library/Application Support/typora-user-images/image-20210306225012886.png)

## K8s 集群中的三种IP(NodeIP、PodIP、ClusterIP)

- Node IP: Node 节点的IP地址，即物理网卡的IP地址
- Pod IP：Pod的IP地址，即docker容器的ip地址，此为虚拟IP地址。
- Cluster IP: Service的IP地址，此为虚拟IP地址。













