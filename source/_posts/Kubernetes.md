---
title: Kubernetes
date: 2019-07-03 17:16:53
updated: 2020-07-03 12:00:00
tags: [Virtualization]
typora-root-url: ../
---

A system for automating deployment, scaling, and management of containerized applications

<!-- more -->

## Components

![](/images/k8s1.png)

### 1. kube-apiserver

> The API server is a component of the Kubernetes control plane that exposes the Kubernetes API. The API server is the front end for the Kubernetes control plane.

一个无状态服务，简单地部署多实例即可均衡负载。

### 2. etcd

强一致性、高可用的分布式Key-value存储系统，满足CAP中的CP，作为K8s所有集群数据的存储。

### 3. kube-scheduler

为新建的pod选择node。考虑的因素主要包括：individual and collective resource requirements, hardware/software/policy constraints, affinity and anti-affinity specifications, data locality, inter-workload interference, and deadlines.

### 4. kube-controller-manager

逻辑上每一个controller都是独立的process。为了方便，把他们编译成一个二进制文件，运行在一个process中。

最常用的Deployments，也是一个built-in的controller。

可在pkg/controller目录下找到内置的controller源码。

### 5. cloud-controller-manager

> The cloud-controller-manager only runs controllers that are specific to your cloud provider. If you are running Kubernetes on your own premises, or in a learning environment inside your own PC, the cluster does not have a cloud controller manager.

### 6. kubelet

> An agent that runs on each node in the cluster. It makes sure that containers are running in a Pod.
>
> The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. The kubelet doesn’t manage containers which were not created by Kubernetes.

### 7. kube-proxy

> kube-proxy is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept.
>
> kube-proxy maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.
>
> kube-proxy uses the operating system packet filtering layer if there is one and it's available. Otherwise, kube-proxy forwards the traffic itself.

使用netfilter，比如iptables

### 8. Container runtime

基本上使用Docker，或者其他Container Runtime Interface的实现。

### 9. addon

主要的两个addon: DNS与Networking。DNS可使用CoreDNS，Networking可使用Flannel、Calico等CNI插件。

## Cluster Networking

> 1. Highly-coupled container-to-container communications: this is solved by pods and localhost communications.
> 2. Pod-to-Pod communications: CNI
> 3. Pod-to-Service communications: this is covered by services.
> 4. External-to-Service communications: this is covered by services.

### 1. Services

> In Kubernetes, a Service is an abstraction which defines a logical set of Pods and a policy by which to access them (sometimes this pattern is called a micro-service).

#### Define a service

```yaml
apiVersion: v1
kind: Service
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

#### Service without a selector

服务的Endpoint指向不在集群内的服务时使用。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

这时候需要手动创建Endpoint指向一个外部地址。

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 192.0.2.42
    ports:
      - port: 9376
```

### 2. Headless Service

> Sometimes you don't need load-balancing and a single Service IP. In this case, you can create what are termed “headless” Services, by explicitly specifying "None" for the cluster IP (spec.clusterIP).

#### with selector

这时候会为pods创建DNS记录

> For headless Services that define selectors, the endpoints controller creates Endpoints records in the API, and modifies the DNS configuration to return records (addresses) that point directly to the Pods backing the Service.

#### Without selectors

> For headless Services that do not define selectors, the endpoints controller does not create Endpoints records. However, the DNS system looks for and configures either:
>
> 1. CNAME records for ExternalName-type Services.
> 2. A records for any Endpoints that share a name with the Service, for all other types.

### 3. ServiceType

- ClusterIP（default）
- NodePort
- LoadBalancer
- ExternalName

> You can also use Ingress to expose your Service. Ingress is not a Service type, but it acts as the entry point for your cluster. It lets you consolidate your routing rules into a single resource as it can expose multiple services under the same IP address.

## Extension Points

> 1. Users often interact with the Kubernetes API using kubectl. Kubectl plugins extend the kubectl binary. They only affect the individual user's local environment, and so cannot enforce site-wide policies.
> 2. The apiserver handles all requests. Several types of extension points in the apiserver allow authenticating requests, or blocking them based on their content, editing content, and handling deletion. These are described in the API Access Extensions section.
> 3. The apiserver serves various kinds of resources. Built-in resource kinds, like pods, are defined by the Kubernetes project and can't be changed. You can also add resources that you define, or that other projects have defined, called Custom Resources, as explained in the Custom Resources section. Custom Resources are often used with API Access Extensions.
> 4. The Kubernetes scheduler decides which nodes to place pods on. There are several ways to extend scheduling. These are described in the Scheduler Extensions section.
> 5. Much of the behavior of Kubernetes is implemented by programs called Controllers which are clients of the API-Server. Controllers are often used in conjunction with Custom Resources.
> 6. The kubelet runs on servers, and helps pods appear like virtual servers with their own IPs on the cluster network. Network Plugins allow for different implementations of pod networking.
> 7. The kubelet also mounts and unmounts volumes for containers. New types of storage can be supported via Storage Plugins.

## Miscellaneous

### 1. Pause container

> The 'pause' container is a container which holds the network namespace for the pod. Kubernetes creates pause containers to acquire the respective pod’s IP address and set up the network namespace for all other containers that join that pod.

其实最主要的作用就是在其他所有容器都死掉的时候，仍然可以保留network namespace的设置。

> The pause container is a container which holds the network namespace for the pod. It does nothing 'useful'. (It's actually just a little bit of assembly that goes to sleep and never wakes up)
>
> This means that your 'apache' container can die, and come back to life, and all of the network setup will still be there.

### 2. Container Network Interface

> Kubernetes’ adoption of the CNI standard allows for many different network solutions to exist within the same ecosystem. The diversity of options available means that most users will be able to find a CNI plugin that suits their current needs and deployment environment, while also providing solutions when their circumstances change. Operating requirements vary immensely between organizations, so having a number of mature solutions with different levels of complexity and feature richness helps Kubernetes satisfy unique requirements while still offering a fairly consistent user experience.