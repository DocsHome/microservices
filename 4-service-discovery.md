# 4、服务发现
本书主要介绍如何使用微服务来构建应用程序，现在是第四章。第一章已经介绍了[微服务架构模式](http://microservices.io/patterns/microservices.html)，并讨论了使用微服务的优点与缺点。第二章和第三章介绍了微服务间通信，并对不同的通信机制作出对比。在本章中，我们将探讨服务发现（service discovery）相关的内容。

## 4.1、为何使用服务发现
我们假设您正在编写某些代码，这些代码调用了有 REST API 或 Thrift API 的服务。为了发送一个请求，您的代码需要知道服务实例的网络位置（IP 地址与端口）。在运行于物理硬件上的传统应用中，服务实例的网络位置是相对静态的。例如，您的代码可以从偶尔更新的配置文件中读取网络位置。

然而，在现代基于云的微服务应用中，这是一个更难解决的问题，如图 4-1 所示。

服务实例具有动态分配的网络位置。此外，由于自动扩展、故障与升级，整组服务实例会动态变更。因此，您的客户端代码需要使用更精确的服务发现机制。

![图 4-1、需要服务寻找帮助的客户端或 API 网关](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/resources/4-1.png)

有两种主要的服务发现模式：客户端发现（client-side discovery）与服务器端发现（server-side discovery）。让我们先来看看客户端发现。

## 4.2、客户端发现模式
当使用[客户端发现模式](http://microservices.io/patterns/client-side-discovery.html)时，客户端负责确定可用服务实例的网络位置和请求负载均衡。客户端查询服务注册中心（service registry），它是可用服务实例的数据库。之后，客户端利用负载均衡算法选择一个可用的服务实例并发出请求。

图 4-2 展示了该模式的结构

![客户端可以承担发现服务任务](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/resources/4-2.png)

服务实例的网络位置在服务注册中心启动时被注册。当实例终止时，它将从服务注册中心中移除。通常使用心跳机制周期性地刷新服务实例的注册。

[Netflix OSS](https://netflix.github.io/) 提供了一个很好的客户端发现模式示例。[Netflix Eureka](https://github.com/Netflix/eureka) 是一个服务注册中心，它提供了一个用于管理服务实例注册和查询可用实例的 REST API。[Netflix Ribbon](https://github.com/Netflix/ribbon) 是一个 IPC 客户端，可与 Eureka 一起使用，用于在可用服务实例之间使请求负载均衡。本章稍后将讨论 Eureka。

客户端发现模式有各种优点与缺点。该模式相对比较简单，除了服务注册中心，没有其他移动部件。此外，由于客户端能发现可用的服务实例，因此可以实现智能的，特定于应用程序的负载均衡决策，比如使用一致性哈希。该模式的一个重要缺点是它将客户端与服务注册中心耦合在一起。您必须为服务客户端使用的每种编程语言和框架实现客户端服务发现逻辑。

现在我们已经了解了客户端发现，接下来让我们看看服务器端发现。

## 4.3、服务端发现模式
服务发现的另一种方式是服务端发现模式。图 4-3 展示了该模式的结构：

![服务器间处理也可以处理服务发现](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/resources/4-3.png)

客户端通过负载均衡器向服务发出请求。负载均衡器查询服务注册中心并将每个请求路由到可用的服务实例。与客户端发现一样，服务实例由服务注册中心注册与销毁。

[AWS Elastic Load Balancer](https://aws.amazon.com/cn/elasticloadbalancing/)（ELB）是一个服务端发现路由示例。ELB 通常用于负载均衡来自互联网的外部流量。然而，您还可以使用 ELB 来负载均衡虚拟私有云（VPC）内部的流量。客户端通过 ELB 使用其 DNS 名称来发送请求（HTTP 或 TCP）。ELB 负载均衡一组已注册的 Elastic Compute Cloud（EC2）实例或 EC2 Container Service（ECS）容器之间的流量。这里没有单独可见的服务注册中心。相反，EC2 实例 与 ECS 容器由 ELB 本身注册。

HTTP 服务器和负载均衡器（如 [NGINX Plus](https://www.nginx.com/products/) 和 NGINX）也可以作为服务端发现负载均衡器。例如，[此博文](https://www.airpair.com/scalable-architecture-with-docker-consul-and-nginx)描述了使用 [Consul Template](https://www.hashicorp.com/blog/introducing-consul-template.html) 动态重新配置NGINX 反向代理。Consul Template 是一个工具，可以从存储在 [Consul 服务注册中心](https://www.consul.io/)中的配置数据中定期重新生成任意配置文件。每当文件被更改时，它都会运行任意的 shell 命令。在博文描述的示例中，Consul Template 会生成一个 nginx.conf 文件，该文件配置了反向代理，然后运行一个命令告知 NGINX 重新加载配置的命令。更复杂的实现可以使用其 [HTTP API 或 DNS](https://www.nginx.com/products/on-the-fly-reconfiguration/) 动态重新配置 NGINX Plus。

某些部署环境（如 [Kubernetes](https://kubernetes.io/) 和 [Marathon](https://mesosphere.github.io/marathon/docs/service-discovery-load-balancing.html)）在群集中的每个主机上运行着一个代理。这些代理扮演着服务端发现负载均衡器角色。为了向服务发出请求，客户端通过代理使用主机的 IP 地址和服务的分配端口来路由请求。然后，代理将请求透明地转发到在集群中某处运行的可用服务实例。

服务端发现模式有几个优点与缺点。该模式的一个很大的优点是发现的细节从客户端抽象出来。客户端只需向负载均衡器发出请求。这消除了为服务客户端使用的每种编程语言和框架都实现发现逻辑的必要性。另外，如上所述，一些部署环境免费提供此功能。然而，这种模式存在一些缺点。除非负载均衡器由部署环境提供，否则它将是您需要设置和管理的另一个高可用系统组件。

## 4.4、服务注册中心
[服务注册中心](http://microservices.io/patterns/service-registry.html)（service registry）是服务发现的一个关键部分。它是一个包含了服务实例网络位置的数据库。服务注册中心必须是高可用和最新的。虽然客户端可以缓存从服务注册中心获得的网络位置，但该信息最终会过期，客户端将无法发现服务实例。因此，服务注册中心由使用了复制协议（replication protocol）来维护一致性的服务器集群组成。

如之前所述，[Netflix Eureka](https://github.com/Netflix/eureka) 是服务注册中心的一个很好例子。它提供了一个用于注册和查询服务实例的 REST API。服务实例使用 POST 请求注册其网络位置。它必须每隔 30 秒使用 PUT 请求来刷新其注册信息。通过使用 HTTP DELETE 请求或实例注册超时来移除注册信息。正如您所料，客户端可以使用 HTTP GET 请求来检索已注册的服务实例。