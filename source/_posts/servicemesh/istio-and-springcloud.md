---
title: Istio对比Springcloud基本分析
date: 2019-10-17 13:49:52
tags:
  - kubernetes
  - k8s
  - istio
categories:
  - Service-Mesh
 
---
   
作者: 李维龙
## 1 Istio 是什么

Istio 是一个完全开源的服务网格，可以降低部署的复杂性，集成了多样化的功能保证微服务架构高效的运行，并提供保护、连接和监控微服务的统一方法。

核心功能

 - 流量管理
      
通过简单的规则配置和流量路由，您可以控制服务之间的流量和 API 调用。Istio 简化了断路器、超时和重试等服务级别属性的配置，并且可以轻松设置 A/B测试、金丝雀部署和基于百分比的流量分割的分阶段部署等重要任务。

 - 安全

Istio 的安全功能使开发人员可以专注于应用程序级别的安全性。Istio 提供底层安全通信信道，并大规模管理服务通信的认证、授权和加密。

 - 可观察性

Istio 强大的追踪、监控和日志记录可以深入了解服务网格部署。通过 Istio 的监控功能，可以真正了解服务性能如何影响上游和下游的功能，而其自定义仪表板可以提供对所有服务性能的可视性，并了解该性能如何影响其他进程。

- 平台支持

Istio 是独立于平台的，旨在运行在各种环境中，包括跨云、内部部署、Kubernetes、Mesos 等。您可以在 Kubernetes 上部署 Istio 或具有 Consul 的 Nomad 上部署。Istio 目前支持：Kubernetes上部署的服务/使用 Consul 注册的服务/虚拟机上部署的服务 

 - 集成和定制

策略执行组件可以扩展和定制，以便与现有的 ACL、日志、监控、配额、审计等方案集成。

## 2 重要组件介绍

Istio是一个开放平台，提供统一的方式来集成微服务，管理跨微服务的流量，执行策略和汇总遥测数据。Istio的控制面板在底层集群管理平台（如Kubernetes，Mesos等）上提供了一个抽象层。

Istio由以下组件组成：

Envoy - 微服务的Sidecar代理，处理集群中的服务间和服务到外部服务的入口/出口流量。代理形成安全的微服务网格，提供丰富的功能，如服务发现，丰富的7层路由，熔断器，策略执行和遥测记录/报告功能。
Mixer - 由代理和微服务使用的集中式组件，用于执行策略，如鉴权，限流，配额，认证，请求跟踪和遥测收集等。

Pilot - 负责在运行时配置代理的组件。

Citadel - 负责证书颁发和轮换的集中式组件

Node Agent - 负责证书颁发和轮换的每节点组件

Galley - 用于在Istio中验证，摄取，聚合，转换和分发配置的中央组件。
Istio目前只支持Kubernetes平台和基于consul的环境

## 3 架构

Istio 服务⽹格逻辑上分为数据平⾯和控制平⾯。

数据平⾯由⼀组以 sidecar ⽅式部署的智能代理（Envoy）组成。这些代理可以调节和控制微服务
及 Mixer 之间所有的⽹络通信。

控制平⾯负责管理和配置代理来路由流量。此外控制平⾯配置 Mixer 以实施策略和收集遥测数据。
  ![architecture of istio](/images/istio-architec.png)

Envoy

  Envoy 被部署为 sidecar，和对应服务在同一个 Kubernetes pod 中。这允许 Istio 将大量关于流量行为的信号作为属性提取出来，而这些属性又可以在 Mixer 中用于执行策略决策，并发送给监控系统，以提供整个网格行为的信息。

Mixer

  Mixer是一个独立于平台的组件，负责在服务网格上执行访问控制和使用策略，并从 Envoy 代理和其他服务收集遥测数据。代理提取请求级属性，发送到 Mixer 进行评估。有关属性提取和策略评估的更多信息，

Pilot

  Pilot 为 Envoy sidecar 提供服务发现功能，为智能路由（例如 A/B 测试、金丝雀部署等）和弹性（超时、重试、熔断器等）提供流量管理功能。它将控制流量行为的高级路由规则转换为特定于 Envoy 的配置，并在运行时将它们传播到 sidecar。

Citadel

  通过内置身份和凭证管理赋能强大的服务间和最终用户身份验证。可用于升级服务网格中未加密的流量，并为运维人员提供基于服务标识而不是网络控制的强制执行策略的能力。从 0.5 版本开始，Istio 支持基于角色的访问控制，以控制谁可以访问您的服务，而不是基于不稳定的三层或四层网络标识。

Galley

  Galley 代表其他的 Istio 控制平面组件，用来验证用户编写的 Istio API 配置。随着时间的推移，Galley 将接管 Istio 获取配置、 处理和分配组件的顶级责任。它负责将其他的 Istio 组件与从底层平台（例如 Kubernetes）获取用户配置的细节隔离开来。


控制平面组件
kubernetes 部署下的控制平面组件如下：

$ kubectl -n istio-system get pod
NAME                                          READY     STATUS 
grafana-5f54556df5-s4xr4                      1/1       Running
istio-citadel-775c6cfd6b-8h5gt                1/1       Running
istio-galley-675d75c954-kjcsg                 1/1       Running
istio-ingressgateway-6f7b477cdd-d8zpv         1/1       Running
istio-pilot-7dfdb48fd8-92xgt                  2/2       Running
istio-policy-544967d75b-p6qkk                 2/2       Running
istio-sidecar-injector-5f7894f54f-w7f9v       1/1       Running
istio-telemetry-777876dc5d-msclx              2/2       Running
istio-tracing-5fbc94c494-558fp                1/1       Running
kiali-7c6f4c9874-vzb4t                        1/1       Running
prometheus-66b7689b97-w9glt                   1/1       Running
 

 

 

## 4 istio系统组件细化到进程级别:

![processes of istio](/images/istio-process.jpg)

说明：

上图是istio组件细化到进程的结构图，在数据平面可以看到istio-ingressgateway、istio-egressgateway 内部实现是一个envoy代理，部署应用程序的pod中包含 istio-init 、user container 、istio-proxy ，其中istio-init 是初始化容器、iptables、路由。

主要的服务：

**Istio-telemetry** 为Envoy提供了数据上报和日志搜集服务，以用于监控告警和日志查询。（高并发下会有性能影响，会间接导致整体性能下降，业界针对这个有较多探讨；为提高性能，需要设置为多实例，防止单点；均衡流量）

**Istio-sidecar-injector** 是sidecar自动注入功能组件，

**Istio-policy** 用于向Envoy提供准入策略控制，黑白名单控制，速率限制等相关策略

**Istio-citadel** 用于安全相关功能，为服务和用户提供认证和鉴权、管理凭据和 RBAC，挂掉则会导致认证，安全相关功能失效，如果部署了 istio-citadel，则 Envoy 每 15 分钟会进行一次重新启动来刷新证书；--set security.enabled=false如果要禁用则通过设置security，它对应的镜像就是citadel

**Istio-galley** istio API配置的校验、各种配置之间统筹，为 Istio 提供配置管理服务，包含有Kubernetes CRD资源的listener，通过用Kubernetes的Webhook机制对Pilot 和 Mixer 的配置进行验证；这个服务挂掉会导致配置校验异常，是一个必须的组件；比如创建gateway、virtualService等资源，就会需要校验；如果不想校验，可以通过设置helm 的选项参数--set global.configValidation=false来关闭校验

**Istio-polit** 控制sidecar中envoy的启动与参数配置；如果异常则envoy无法正常启动，应用服务的流量无法进行拦截和代理，所有配置、流量规则、策略无法生效。

 

资料：

[Istio knowledge map 知识图谱](https://github.com/servicemesher/istio-knowledge-map)
 

仓库

Istio项目分为多个GitHub库。

istio/istio： 这是Istio的主要存储库。 它托管了Istio的核心组件，以及管理Istio开源项目的示例程序和各种文档。 这包括：

security：此目录包含与安全性相关的代码，包括Citadel（充当证书颁发机构），node agent。

pilot：此目录包含特定于平台的代码，用于填充抽象服务模型，在应用程序拓扑更改时动态重新配置代理，以及将路由规则转换为特定于代理的配置。

istioctl：该目录包含istioctl命令行实用程序的代码。

mixer：此目录包含用于对通过代理的流量实施各种策略以及从代理和服务收集遥测数据的代码。 有插件可与各种云平台，策略管理服务和监控服务连接。

istio/API：该存储库定义了Istio平台的组件级API和通用配置格式。

istio/proxy： Istio代理包含Envoy代理的扩展（以Envoy过滤器的形式），允许代理将策略执行决策委托给Mixer。



## 5 Istio 安装 

下载 istio 安装包 

curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.2.5 sh -

配置环境变量 PATH

k8s 安装，进入 安装包目录 ，执行：for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl apply -f $i; done

选择模式 ：

宽容模式 mutual TLS： kubectl apply -f install/kubernetes/istio-demo.yaml

严格模式 mutual TLS：kubectl apply -f install/kubernetes/istio-demo-auth.yaml

 

Istio + k8s部署结构图  

![k8s+istio结构图](/images/k8s+istio.jpg)

说明：

从图中可以看出，Istio 做服务治理，K8s与istio整合，K8s做容器编排，微服务是以Docker容器的方式运行
 

## 6 用户请求访问

![用户请求在istio路由流程](/images/request-gateway-istio.png)

说明：

(1) 用户根据指定网络地址对Service X发起请求，通过外层网关服务public Ingress gateway 根据指定规则路由到目标pod，request 被代理sidecar拦截，sidecar与Service X 之间互相通信，如果在Service X 执行请求的过程中，调用了Service Y，在Service X发送调用请求后，请求会被sidecar拦截并解析，然后与Service Y的sidecar进行通信，Service Y再与它的sidecar通信，最后响应请求结束。

(2) 当service mesh 中的服务需要去调用网格外的服务时，通过设置Egress gateway ，设置出口网关，路由到指定的外部服务。       



服务间调用

调用配置：   
 ![服务之间调用url配置](/images/invoke-service-config.png)


 

调用流程图：

![服务之间调用流程](/images/invoke-service-flow.png)         

说明：

Service A 请求调用Service B,通过服务名在service name System 中获取到满足指定策略的服务地址（Ip+端口），service name System 可以当成是一个注册中心，istio中pilot 组件充当注册中心发现服务，当获取到服务的地址，就可以直接去请求Service B 的某一个节点，最后返回响应结果。



## 7 总结

通过了解istio 实现service mesh，对比SpringCloud可以发现，istio是将服务的治理下沉到了架构层面，在我们的服务或应用程序中不需要再去考虑服务治理相关操作，这也很好的将开发和运维剥离，同时在服务治理的层面，istio实现的粒度更加的精确，方式更加的便捷，不同于SpringCloud中我们需要在服务的配置中修改，并重启服务。springcloud 在负载均衡和熔断处理，一般都是在客户端处理，也就是调用端，而istio在负载均衡和熔断处理时，是通过Envoy代理实现。

istio与k8s结合使用时，istio 服务会在每个kubelet都启用，应用部署的时候可以选择指定node，也可以指定资源条件启动，k8s会去根据自身算法寻找满足条件的node，当没有满足的node时，会启动报错，被踢除。一个 pod 可以包含一个或者多个容器，但通常是一个。而使用 Istio 的时候，需要在 pod 中主容器旁注入一个 sidecar，也就是上面提到的代理（Envoy）。

最后我们通过从服务发现、负载均衡、熔断处理 、网关路由 四个方面对istio和SpringCloud做了对比：

  （1）服务发现：SpringCloud 使用Eureka等注册中心，实现服务发现，应用的服务必须都指定注册中心的地址进行注册，发现服务时，通过注册中心去寻址解析到相对应得服务地址，进行请求；istio 在低版本时支持eureka 注册中心，高版本已经去除eureka，所以目前发现服务有两种方式，一种是使用k8s api-server，另一种是Consul，istio与k'8s结合的比较好，所以可以直接使用k8s的服务发现api-server，istio中的pilot组件就是和k8s api-server进行通信，实现服务发现，维护服务列表；从服务发现的层面去比较，可以看到 istio 架构中，开发不需要去考虑服务发现的过程，不需要将服务注册到指定的注册注册中心。

  （2）负载均衡：SpringCloud 支持负载均衡的组件Ribbon、Feign，我们需要在代码中使用@LoadBalanced 注解开启负载均衡客户端，自定义负载均衡策略可以在配置文件中进行配置；istio 中Envoy组件配置负载均衡策略实现转发请求，当前支持的主要负载均衡算法：轮询、随机、最小连接数。通过比较可以发现，两者负载均衡的工作流程类似，但是istio 负载均衡策略配置的粒度要更小，类型更多。

  （3）熔断处理：SpringCloud 熔断组件Hystrix ，可以在配置中进行开启并相关配置，并实现回调方法；istio利用Envoy网格，Envoy 在网络级别强制实现断路，而不必为每个应用程序单独配置或编程。

  （4）网关：SpringCloud 有gateway网关组件，我们可以在实现组件的服务，利用过滤器和拦截器实现自定义的网关路由；istio 可以通过gateway配置ingress，并配置相关的virtualService，来实现网关路由转发。istio gateway可以实现的粒度更小，可以根据角色、用户等更小的粒度进行路由。

 

demo 代码地址：https://github.com/WeiLong-lee/istio-study.git

istio 官网文档地址：https://istio.io/docs/  