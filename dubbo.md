# Dubbo

## 概述

Dubbo是一款高性能、轻量级的开源Java RPC框架，它提供了三大核心能力：面向接口的远程方法调用，智能容错和负载均衡，以及服务自动注册和发现。


## 架构体系

- dubbo-common:公共逻辑模块: 包括Util类和通用模型
- dubbo-remoting 远程通信模块: 相当于dubbo协议的实现，如果RPC使用RMI协议则不需要使用此包
- dubbo-rpc 远程调用模块: 抽象各种协议，以及动态代理，包含一对一的调用，不关心集群的原理。
- dubbo-cluster 集群模块: 将多个服务提供方伪装成一个提供方,包括负载均衡,容错,路由等,集群的地址列表可以是静态配置的,也可以是注册中心下发的.
- dubbo-registry 注册中心模块: 基于注册中心下发的集群方式,以及对各种注册中心的抽象
- dubbo-monitor 监控模块: 统计服务调用次数,调用时间,调用链跟踪的服务.
- dubbo-config 配置模块: 是dubbo对外的api,用户通过config使用dubbo,隐藏dubbo所有细节
- dubbo-container 容器模块: 是一个standlone的容器,以简单的main加载spring启动,因为服务通常不需要Tomcat/Jboss等web容器的特性,没必要用web容器去加载服务