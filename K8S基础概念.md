



生成yaml 文件

日志管理

监控管理

Kubernetes Handbook——Kubernetes 中文指南/云原生应用架构实践手册

https://jimmysong.io/kubernetes-handbook/

knative

https://www.servicemesher.com/getting-started-with-knative/knative-overview.html

istio service mesh

https://www.servicemesher.com/istio-handbook/concepts/overview.html



## Kubernetes 架构和组件

 Kubernetes 利用了 “期望状态” 原则。就是说，你定义了组件的期望状态，而 Kubernetes 要将它们始终调整到这个状态。

### 集群

所有其他组件都是集群的一部分。你也可以创建多个虚拟集群，称为命名空间 (namespace)，它们是同一个物理集群的一部分。这与你可以在同一物理服务器上创建多个虚拟机的方式非常相似。如果你不需要，也没有明确定义的命名空间，那么你的集群将在始终存在的默认命名空间中创建。

### 节点 (node)

节点是集群中的单个机器

有 2 种类型的节点master 节点和 worker 节点

主节点是一个控制其他所有节点的特殊节点，它向集群中的所有其他节点发送消息，将工作分配给它们，工作节点向主节点上的 API Server 汇报。

Woker 节点是 Kubernetes 中真正干活的节点，当你在应用中部署容器或 pod（稍后定义）时，其实是在将它们部署到 worker 节点上运行。Worker 节点托管和运行一个或多个容器的资源。

### pod

Kubernetes 中的逻辑而非物理的工作单位称为 pod。 

一个 pod 允许你把多个容器，并指定它们如何组合在一起来创建应用程序。

一个 Kubernetes pod 通常包含一个或多个 Docker 容器，所有的容器都作为一个单元来管理。

### service

Kubernetes 中的 service 是一组逻辑上的 pod。

把一个 service 看成是一个 pod 的逻辑分组，它提供了一个单一的 IP 地址和 DNS 名称，你可以通过它访问服务内的所有 pod

### ReplicationController

实际管理 pod 生命周期的组件 ，ReplicationController 有助于实现我们所期望的指定运行的 pod 数量的状态。

# Kubectl

kubectl 是一个命令行工具，用于与 Kubernetes 集群和其中的 pod 通信。使用它你可以查看集群的状态，列出集群中的所有 pod，进入 pod 中执行命令等。你还可以使用 YAML 文件定义资源对象，然后使用 kubectl 将其应用到集群中。

## Kubernetes 中的自动扩展

因为 Kubernetes 能够自动扩展应用实例的数量以满足工作负载的需求。

##  kubernetes Ingress 和 Egress？

进入 Kubernetes pod 的流量称为 Ingress，而从 pod 到集群外的出站流量称为 egress。我们创建入口策略和出口策略的目的是限制不需要的流量进入和流出服务。而这些策略也是定义 pod 使用的端口来接受传入和传输传出数据 / 流量的地方。

## Ingress Controller

但是在定义入口和出口策略之前，你必须首先启动被称为 Ingress Controller（入口控制器）的组件；这个在集群中默认不启动。有不同类型的入口控制器，Kubernetes 项目默认只支持 Google Cloud 和开箱即用的 Nginx 入口控制器。通常云供应商都会提供自己的入口控制器。

## Replica 和 ReplicaSet

为了保证应用程序的弹性，需要在不同节点上创建多个 pod 的副本。这些被称为 Replica

举例来说，如果你目前有 2 个 pod 的副本，而你所希望的状态应该有 3 个，那么 Replication Controller 或 ReplicaSet 会自动检测到这个要求，并指示 Deployment Controller 根据预定义的设置部署一个新的 pod。

## 服务网格

[服务网格 (Service Mesh)](https://jimmysong.io/blog/what-is-a-service-mesh/) 用于管理服务之间的网络流量，是云原生的网络基础设施层，也是 [Kubernetes 次世代的云原生应用](https://jimmysong.io/blog/post-kubernetes-era/) 的重要组成部分。

服务网格利用容器之间的网络设置来控制或改变应用程序中不同组件之间的交互。

