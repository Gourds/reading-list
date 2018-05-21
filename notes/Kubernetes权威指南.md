## Kubernetes学习笔记

K8s是google久负盛名的内部大规模集群管理系统Borg的一个开源版本，Borg基于容器技术，目的是实现资源管理自动化，以及跨多个数据中心的资源利用率最大化。

K8s是一个完备的分布式系统支撑平台。具有以下优点
- 完备的集群管理能力（包括多层次的安全防护和准入机制）
- 多租户应用支撑能力
- 透明的服务注册和服务发现机制
- 内建负载均衡器、强大的故障发现和自我修复能力
- 服务滚动升级和在线扩容能力
- 可扩展的资源自动调度机制及多粒度资源配额管理能力
- 完善的管理工具（包括开发、部署测试、运维监控）

## 主要概念

Master：集群控制节点
- kubernetes API Server(kube-apiserver),提供HTTP Rest接口的关键服务进程，是k8s中所有增删改查操作的唯一入口，也是集群控制的入口进程
- kubernets Controller Manager(kube-controller-manager),Kubernetes里所有资源对象的自动化控制中心
- kubernets Scheduler(kube-scheduler),负责资源调度（Pod调度）的进程
- Etcd进程（一般在master上可以分离成集群出去），存放kubernets中所有资源对象的数据

Node：k8s集群中工作负载节点
- kubelet: 负责pod对应的容器的创建、启停等任务，与master节点密切协作，实现集群管理的基本功能
- kube-proxy：实现Kubernets Service的通信及负载均衡机制的重要组件
- Docker Engine： 即安装docker，负责容器的创建管理工作

Pod: pod是k8s最重要也是最基本的概念
- Pause容器：根容器，pause容器对应的镜像属于k8s平台的一部分，每个pod有且只有一个，主要作用为判断容器存活状态及业务容器通信问题（同Pod中容器共用Pause容器的IP及Volume）
- conainter：用户容器，即业务容器，可以是一个或者多个
- Endpoint： pod的IP加上容器端口(containerPort)叫做endpoint，一个pod可能存在一个或多个endpoint

## 基础命令
``` bash
kubectl create -f xxx-rc.yaml #创建RC 执行创建的类型由yaml内容中`kind`控制
kubectl get nodes #查看节点数量情况
kubectl describe node kubernetes-n1 #查看节点详细信息
kubectl get rc #查看RC(Replication Controller)
kubectl get pods #查看pods的创建情况
kubectl get svc #查看service情况
kubectl get deployments #查看deploy信息
kubectl get rs #查看Replica Set
kubectl describe deployments #查看deployment控制pod水平扩展的过程
kubectl describe pod mysql #查看具体pod的情况
kubectl scale rc myweb --replicas=1 #动态调整service的pod数量
kubectl delete pod mysql-6sw6m #删除一个指定pod
kubectl delete svc mysql #删除一个服务
kubectl delete rc mysql #删除一个rc
```

