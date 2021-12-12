---
layout: post
title:  "Kubernetes集群搭建"
date:   2021-08-12 14:02:23
comments: false
categories: 自动化
tag: 自动化
description: Kubernetes集群搭建,比较简略。                                                        
---
* content
{:toc}
#### introduction

`Kubernetes`是 `Google` 开源的容器集群管理系统，基于 `Docker` 构建一个容器的调度服务，提供资源调度、均衡容灾、服务注册、动态扩缩容等功能套件。 

## 1 Kubernetes

#### 1.`1`概述

`Kubernetes`是 `Google` 开源的容器集群管理系统，基于 `Docker` 构建一个容器的调度服务，提供资源调度、均衡容灾、服务注册、动态扩缩容等功能套件。 `Kubernetes` 基于 `docker` 容器的云平台，简写成： k8s。

#### 1.`2` `kubeadm`、`kubelet` 和 `kubectl`

`kubeadm`：用来初始化集群的指令。

`kubelet`：在集群中的每个节点上用来启动 `Pod` 和容器等。

kubectl：用来与集群通信的命令行工具。

## 2.搭建

#### 2.`1` 搭建思路

搭建过程本人不想写的过于具体,因为版本和系统安装方式有差异，越具体意味着局限性越大,所以只把过程大概需要做什么写出来,然后贴一些官方文档。

#### 2.`2` 搭建步骤

(1) 安装`docker` 或者支持`CRI`的容器技术，例如cri-o，cri-containerd，rkt等等。

```
    yum -y install docker-ce

    systemctl enable docker && systemctl start docker
```

(2) 禁用swap

```
      1.swapoff -a
      2.# 注释 swap 行
         `vim` /`etc`/`fstab`

      `3`.查看  `free`
```

(3) 禁用SELinux

 将 SELinux 设置为 permissive 模式（相当于将其禁用）
 
```
 sudo setenforce 0 sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

(4)安装`kubeadm`,`kubelet`,`kubectl`不同操作系统可以参见`https`://`kubernetes`.`io`/`zh`/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

```
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

sudo systemctl enable --now kubelet
```

(5) 初始化主节点

```
echo "1" >>/proc/sys/net/bridge/bridge-nf-call-iptables 
```

真正初始化前,建议看一下`Cgroup` 驱动程序`cgroupfs` `systemd`区别，我们要保证`docker`和`kubelet`的`Cgroup` 驱动程序一致。`https`://kubernetes.io/zh/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/

```
kubeadm init --config kubeadm-config.yaml
```

(6)初始化成功之后我们就可以拿下图命令去加入节点了。

![](https://bo07997.github.io/myBlog/styles/images/Blog/Kubernetes集群搭建/1.png)


(7) 初始化node(普通节点)

  1.先在普通节点执行(1)-(4)

  2.执行下面命令,黄色部分可选,但是建议给节点命名

```
kubeadm join 10.17.2.52:6443 --token micm5r.0q4ziy8zweeazq9x \
--discovery-token-ca-cert-hash sha256:4076884a92d63b29e19f8b67167a481d9875bcd36383187d7539e14cee705e38 --node-name 10.18.8.8 --v=5
```

(8) 成功后我们在master节点执行kubectl get nodes,但是节点却显示Not Ready ,因为网络组建还没启动。

#### 2.3 组建网络(calico)

`Calico` 简介

`Calico`

是一种容器之间互通的网络方案 。在虚拟化平台中，比如 `OpenStack` 、 `Docker` 等都需要
实现 `workloads` 之间互连，但同时也需要对容器做隔离控制 。而在多数的虚拟化平台实现中，通常都
使用 二层隔离技术来实现容器的网络 ，这些二层的技术有一些弊端，比如需要依赖 `VLAN` 、 `bridge` 和
隧道等技术，其中 `bridge` 带来了复杂性， `vlan` 隔离和 `tunnel` 隧道在 拆包或加包头时，则消耗更多的
资源并对物理环境也有要求 。 随着网络规模的增大，整体会变得越加复杂。

`flannel`

方案： 需要在每个节点上把发向容器的数据包进行封装后，再用隧道将封装后的数据包发
送到运行着目标 `Pod` 的 `node` 节点上。目标 `node` 节点再负责去掉封装，将去除封装的数据包发送到目
标 `Pod` 上。数据通信性能则大受影响

`Overlay`

方案： 在下层主机网络的上层，基于隧道封装机制，搭建层叠网络，实现跨主机的通信；
Overlay 无疑是架构最简单清晰的网络实现机制，但数据通信性能则大受影响。


这里本人了解得也不是特别多,只是遇到网络不通的情况下才开始重视起来,有空可以研究一下。



我们这里只需要一行代码搞定

```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

至此所有节点初始化完毕,这是我在测试环境搭建的截图。

![](https://bo07997.github.io/myBlog/styles/images/Blog/Kubernetes集群搭建/2.png)


#### 2.4 常用命令及排查思路

查看命令

  `1`.`kubectl` `get` `deployment` -`A`       查看`deployment` 



  `2`.`kubectl` `get` `node` -`A`       查看节点

  `3`.`kubectl` `get` `pod` -`A`       查看`pod`



描述命令

  `1`.`kubectl` `describe`  `node` `nodeName`  描述一个节点

  `2`. `kubectl` `describe`  `pod` `podName`  描述一个`pod`



删除命令

  `1`. `kubectl` `delete` `pod` `podName`  删除一个`pod`资源



其它常用命令

  `1`.`kubectl` `exec` -`it` `altraserver` -- /`bin`/`sh`  进入容器

 

系统命令

  `1`.`systemctl` `status` `kubelet`服务    查看`kubelet`服务状态

  `2`.`journalctl` -`xefu` `kubelet`服务  查看`kubelet`服务日志



排查思路

`1`.用`get` 命令查看资源对象是否正确。

`2`.用描述命令去查看具体情况。

`3`.如果有报错一般可以用关键字去判断了。

`4`.如果是`kebelet`之类的服务，可以用系统命令查看日志,`journalctl` -`xefu` `kubelet` ，`systemctl` `status` `kubelet`前者更为详细和实时。

`5`.如果是`pod`,可以直接去到宿主机,通过`docker` `logs`查看日志,也可以通过`kubectl` `logs`查看日志。
