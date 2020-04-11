---
title: Consul学习
tags:
  - Consul
categories:
  - Dev
  - Frame
date: 2019-11-19 08:43:00
cover: true
---

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/Jennifer-3.png)
<!-- more -->

服务注册与服务发现是在分布式服务架构中常常会涉及到的东西，业界常用的服务注册与服务发现工具有 [ZooKeeper](https://zookeeper.apache.org/)、[etcd](https://coreos.com/etcd/)、[Consul](https://www.consul.io/) 和 [Eureka](https://github.com/Netflix/eureka)。Consul 的主要功能有服务发现、健康检查、KV存储、安全服务沟通和多数据中心。Consul 与其他几个工具的区别可以在这里查看 [Consul vs. Other Software](https://www.consul.io/intro/vs/index.html)。
##为什么需要有服务注册与服务发现？
假设在分布式系统中有两个服务 Service-A （下文以“S-A”代称）和 Service-B（下文以“S-B”代称），当 S-A 想调用 S-B 时，我们首先想到的时直接在 S-A 中请求 S-B 所在服务器的 IP 地址和监听的端口，这在服务规模很小的情况下是没有任何问题的，但是在服务规模很大每个服务不止部署一个实例的情况下是存在一些问题的，比如 S-B 部署了三个实例 S-B-1、S-B-2 和 S-B-3，这时候 S-A 想调用 S-B 该请求哪一个服务实例的 IP 呢？还是将3个服务实例的 IP 都写在 S-A 的代码里，每次调用 S-B 时选择其中一个 IP？这样做显得很不灵活，这时我们想到了 Nginx 刚好就能很好的解决这个问题，引入 Nginx 后现在的架构变成了如下图这样：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0xYjBhZjFhOTgyNGIyMGRjLnBuZw?x-oss-process=image/format,png )

引入 Nginx 后就解决了 S-B 部署多个实例的问题，还做了 S-B 实例间的负载均衡。但现在的架构又面临了新的问题，分布式系统往往要保证高可用以及能做到动态伸缩，在引入 Nginx 的架构中，假如当 S-B-1 服务实例不可用时，Nginx 仍然会向 S-B-1 分配请求，这样服务就不可用，我们想要的是 S-B-1 挂掉后 Nginx 就不再向其分配请求，以及当我们新部署了 S-B-4 和 S-B-5 后，Nginx 也能将请求分配到 S-B-4 和 S-B-5，Nginx 要做到这样就要在每次有服务实例变动时去更新配置文件再重启 Nginx。这样看似乎用了 Nginx 也很不舒服以及还需要人工去观察哪些服务有没有挂掉，Nginx 要是有对服务的健康检查以及能够动态变更服务配置就是我们想要的工具，这就是服务注册与服务发现工具的用处。下面是引入服务注册与服务发现工具后的架构图：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS03ZTQxZmM4ZGQ5YTc0NmIxLnBuZw?x-oss-process=image/format,png)

在这个架构中：

* 首先 S-B 的实例启动后将自身的服务信息（主要是服务所在的 IP 地址和端口号）注册到注册工具中。不同注册工具服务的注册方式各不相同，后文会讲 Consul 的具体注册方式。
*  服务将服务信息注册到注册工具后，注册工具就可以对服务做健康检查，以此来确定哪些服务实例可用哪些不可用。
* S-A 启动后就可以通过服务注册和服务发现工具获取到所有健康的 S-B 实例的 IP 和端口，并将这些信息放入自己的内存中，S-A 就可用通过这些信息来调用 S-B。
* S-A 可以通过监听（Watch）注册工具来更新存入内存中的 S-B 的服务信息。比如 S-B-1 挂了，健康检查机制就会将其标为不可用，这样的信息变动就被 S-A 监听到了，S-A 就更新自己内存中 S-B-1 的服务信息。

所以务注册与服务发现工具除了服务本身的服务注册和发现功能外至少还需要有健康检查和状态变更通知的功能。

## Consul内部原理

Consul 作为一种分布式服务工具，为了避免单点故障常常以集群的方式进行部署，在 Consul 集群的节点中分为 Server 和 Client 两种节点（所有的节点也被称为Agent），Server 节点保存数据，Client 节点负责健康检查及转发数据请求到 Server；Server 节点有一个 Leader 节点和多个 Follower 节点，Leader 节点会将数据同步到 Follower 节点，在 Leader 节点挂掉的时候会启动选举机制产生一个新的 Leader。

Client 节点很轻量且无状态，它以 RPC 的方式向 Server 节点做读写请求的转发，此外也可以直接向 Server 节点发送读写请求。下面是 Consul 的架构图：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1mNWJlYzdiYmFlYWVhZDY3LnBuZw?x-oss-process=image/format,png)

首先Consul支持多数据中心，在上图中有两个DataCenter，他们通过Internet互联，同时请注意为了提高通信效率，只有Server节点才加入跨数据中心的通信。

在单个数据中心中，Consul分为Client和Server两种节点（所有的节点也被称为Agent），Server节点保存数据，Client负责健康检查及转发数据请求到Server；Server节点有一个Leader和多个Follower，Leader节点会将数据同步到Follower，Server的数量推荐是3个或者5个，在Leader挂掉的时候会启动选举机制产生一个新的Leader。

集群内的Consul节点通过gossip协议（流言协议）维护成员关系，也就是说某个节点了解集群内现在还有哪些节点，这些节点是Client还是Server。单个数据中心的流言协议同时使用TCP和UDP通信，并且都使用8301端口。跨数据中心的流言协议也同时使用TCP和UDP通信，端口使用8302。

集群内数据的读写请求既可以直接发到Server，也可以通过Client使用RPC转发到Server，请求最终会到达Leader节点，在允许数据轻微陈旧的情况下，读请求也可以在普通的Server节点完成，集群内数据的读写和复制都是通过TCP的8300端口完成。

## Consul 的主要特点

`Service Discovery` : 服务注册与发现，Consul 的客户端可以做为一个服务注册到 Consul，也可以通过 Consul 来查找特定的服务提供者，并且根据提供的信息进行调用。

`Health Checking`: Consul 客户端会定期发送一些健康检查数据和服务端进行通讯，判断客户端的状态、内存使用情况是否正常，用来监控整个集群的状态，防止服务转发到故障的服务上面。

`KV Store`: Consul 还提供了一个容易使用的键值存储。这可以用来保持动态配置，协助服务协调、建立 Leader 选举，以及开发者想构造的其它一些事务。

`Secure Service Communication`: Consul 可以为服务生成分布式的 TLS 证书，以建立相互的 TLS 连接。 可以使用 intentions 定义允许哪些服务进行通信。 可以使用 intentions 轻松管理服务隔离，而不是使用复杂的网络拓扑和静态防火墙规则。

`Multi Datacenter`: Consul 支持开箱即用的多数据中心，这意味着用户不需要担心需要建立额外的抽象层让业务扩展到多个区域。

`Consul 角色`
* Server: 服务端, 保存配置信息, 高可用集群, 在局域网内与本地客户端通讯, 通过广域网与其它数据中心通讯。 每个数据中心的 Server 数量推荐为 3 个或是 5 个。
* Client: 客户端, 无状态, 将 HTTP 和 DNS 接口请求转发给局域网内的服务端集群。

Consul 旨在对 DevOps 社区和应用程序开发人员友好，使其成为现代、弹性基础架构的理想选择。

## Consul 的调用过程

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1iZGFmM2VjOThjMTk5ZWQwLnBuZw?x-oss-process=image/format,png )

1、当 Producer 启动的时候，会向 Consul 发送一个 post 请求，告诉 Consul 自己的 IP 和 Port；

2、Consul 接收到 Producer 的注册后，每隔 10s（默认）会向 Producer 发送一个健康检查的请求，检验 Producer 是否健康；

3、当 Consumer 发送 GET 方式请求 /api/address 到 Producer 时，会先从 Consul 中拿到一个存储服务 IP 和 Port 的临时表，从表中拿到 Producer 的 IP 和 Port 后再发送 GET 方式请求 /api/address；

4、该临时表每隔 10s 会更新，只包含有通过了健康检查的 Producer。

## Consul 和 eureka的对比

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1hZTEyNjI5Mzk4OTgxMmQxLnBuZw?x-oss-process=image/format,png )

## Consul服务发现原理

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS00MWJhNTZkMWU1NGQ3YzA4LmpwZw?x-oss-process=image/format,png)

首先需要有一个正常的Consul集群，有Server，有Leader。这里在服务器Server1、Server2、Server3上分别部署了Consul Server，假设他们选举了Server2上的Consul Server节点为Leader。这些服务器上最好只部署Consul程序，以尽量维护Consul Server的稳定。

然后在服务器Server4和Server5上通过Consul Client分别注册Service A、B、C，这里每个Service分别部署在了两个服务器上，这样可以避免Service的单点问题。服务注册到Consul可以通过HTTP API（8500端口）的方式，也可以通过Consul配置文件的方式。Consul Client可以认为是无状态的，它将注册信息通过RPC转发到Consul Server，服务信息保存在Server的各个节点中，并且通过Raft实现了强一致性。

最后在服务器Server6中Program D需要访问Service B，这时候Program D首先访问本机Consul Client提供的HTTP API，本机Client会将请求转发到Consul Server，Consul Server查询到Service B当前的信息返回，最终Program D拿到了Service B的所有部署的IP和端口，然后就可以选择Service B的其中一个部署并向其发起请求了。如果服务发现采用的是DNS方式，则Program D中直接使用Service B的服务发现域名，域名解析请求首先到达本机DNS代理，然后转发到本机Consul Client，本机Client会将请求转发到Consul Server，Consul Server查询到Service B当前的信息返回，最终Program D拿到了Service B的某个部署的IP和端口。
