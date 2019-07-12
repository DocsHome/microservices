# 6、选择部署策略

本书主要介绍关于如何使用微服务构建应用，这是本书的第六章。第一章介绍了[微服务架构模式](http://microservices.io/patterns/microservices.html)，讨论了使用微服务的优点与缺点。之后的章节讨论了微服务架构的方方面面：[使用 API 网关](2-using-an-api-gateway.md)、[进程间通信](3-inter-process-communication.md)、[服务发现](4-service-discovery.md)和[事件驱动数据管理](5-event-driven-data-management-for-microservices.md)。在本章中，我们将介绍部署微服务的策略。

<a id="motivations"></a>

## 6.1、动机

部署[单体应用](http://microservices.io/patterns/monolithic.html)意味着运行一个或多个来自单个较大应用的相同副本。你通常会配置 N 个服务器（物理或虚拟），每台服务器上会运行 M 个应用实例。单体应用的部署并不总是那么简单，但它比部署微服务应用要简单得多。

[微服务应用](http://microservices.io/patterns/microservices.html)由数十甚至上百个服务组成。服务使用不同的语言和框架编写。每个服务都是一个迷你应用，有自己特定的部署、资源、扩展和监视要求。例如，你需要根据服务的需求为每个服务运行一定数量的实例。此外，必须为每个服务实例提供相应的 CPU、内存和 I/O 资源。更有挑战性的是尽管这些如此复杂，部署服务也必须要快速、可靠和具有成本效益。

有几种不同的微服务部署模式。我们首先看看单主机多服务实例模式。

<a id="multiple-service-instances-per-host-pattern"></a>

## 6.2、单主机多服务实例模式

一种微服务的部署方式是使用[单主机多服务实例](http://microservices.io/patterns/deployment/multiple-services-per-host.html)（Multiple Service Instances per Host）模式。当使用此模式时，你可以提供一台（个）或多个台（个）物理主机或虚拟主机，并在每台（个）主机上运行多个服务实例。从多方面来讲，这是传统的应用部署方式。每个服务实例在一台（个）或多台（个）主机的标准端口上运行。你需要[像照顾宠物一样照顾这些主机](https://www.nginx.com/blog/microservices-at-netflix-architectural-best-practices/#stateless-servers)。

图 6-1 展示了该模式的结构：

![图 6-1、主机可支持多个服务实例](resources/6-1.png)

这种模式扩展出了多个变体。一个变体是每个服务实例都是一个进程或进程组。你可以将一个 Java 服务实例部署到 [Apache Tomcat](http://tomcat.apache.org/) 服务器作为一个 web 应用。一个 [Node.js](https://nodejs.org/) 服务实例可能包含一个父进程和一个或多个子进程。

此模式的另一个变体是在同一进程或进程组中运行多个服务实例。你可以在同一个 Apache Tomcat 服务器上部署多个 Java Web 应用，或在同一 OSGI 容器中运行多个 OSGI 软件包。

单主机多服务实例模式有优点也有缺点。主要优点是它的资源使用率相对较高。多个服务实例共享服务器及操作系统。如果进程或进程组运行了多个服务实例（例如，多个 Web 应用共享相同的 Apache Tomcat 服务器和 JVM），则效率更高。

这种模式的另一个优点是部署服务实例相对较快。你只需将服务复制到主机并启动它。如果服务是使用 Java 编写，则可以复制 JAR 或 WAR 文件。对于其他语言，例如 Node.js 或 Ruby，你可以直接复制源代码。

另外，由于开销较小，启动一个服务是非常快的。如果服务有自己的进程，你只需要启动它即可。如果服务是同一容器进程或进程组中运行的几个实例之一，则可以将其动态部署到容器中或者重新启动容器。

尽管这很有吸引力，但单主机多服务实例模式有一些明显的缺点。一个主要的缺点是服务实例很少隔离或没有隔离，除非每个服务实例是一个单独的进程。虽然你可以准确地监控每个服务实例的资源利用率，但你不能限制每个实例使用的资源占比。一个行为不当的服务实例可能会占用掉主机的所有内存或 CPU。

如果多个服务实例在同一进程中运行，那么将毫无隔离可言。例如，所有实例可能共享相同的 JVM 堆。行为不当的服务实例可能会轻易地破坏在同一进程中运行的其他服务。此外，你无法监控每个服务实例使用的资源。

这种方式的另一个重要问题是部署服务的运维团队必须了解部署的具体细节。服务可以是用不同的语言和框架编写，因此开发团队必须与运维交代许多细节。这种复杂性无疑加大了部署过程中的错误风险。

正如你所见，尽管这种方式很简单，但单主机多服务实例模式确实存在一些明显的缺点。现在让我们来看看可以绕过这些问题的其他部署方式。

<a id="service-instance-per-host-pattern"></a>

## 6.3、每个主机一个服务实例模式

部署微服务的另一种方式是使用[每个主机一个服务实例](http://microservices.io/patterns/deployment/single-service-per-host.html)（Service Instance per Host）模式。当使用此模式时，你可以在主机上单独运行一个服务实例。这种模式有两种不同形式：每个虚拟机一个服务实例模式和每个容器一个服务实例模式。

<a id="service-instance-per-virtual-machine-pattern"></a>

### 6.3.1、每个虚拟机一个服务实例模式

当你使用[每个虚拟机一个服务实例](http://microservices.io/patterns/deployment/service-per-vm.html)模式时，将每个服务打包成一个虚拟机（VM）镜像（如 [Amazon EC2 AMI](https://aws.amazon.com/cn/ec2/)）。每个服务实例都是一个使用该 VM 镜像启动的 VM（例如，一个 EC2 实例）。

图 6-2 展示了该模式的结构：

![图 6-2、服务可以各自运行在自己的虚拟机中](resources/6-2.png)

这是 Netflix 部署其视频流服务的主要方式。Netflix 使用 [Aminator](https://github.com/Netflix/aminator) 将每个服务打包成 EC2 AMI。每个运行的服务实例都是一个 EC2 实例。

你可以使用多种工具来构建自己的虚拟机。你可以配置你的持续集成（CI）服务器（比如 [Jenkins](https://jenkins.io/index.html)）来调用 Aminator 将服务打包为一个 EC2 AMI。另一个选择是使用 [Packer](https://www.packer.io/) 自动化创建虚拟机镜像。与 Aminator 不同，它支持各种虚拟化技术，包括 EC2、DigitalOcean、VirtualBox 和 VMware。

[Boxfuse](https://boxfuse.com/) 公司有一种非常好的方式用来构建虚拟机镜像，其克服了我将在下面提到的虚拟机的缺点。Boxfuse 将 Java 应用打包成一个最小化的 VM 镜像。这些镜像可被快速构建、快速启动且更加安全，因为它们只暴露一个有限的攻击范围。

[CloudNative](https://cloudnative.io/) 公司拥有 Bakery，这是一个用于创建 EC2 AMI 的 SaaS 产品。你可以配置你的 CI 服务器，在微服务通过测试后调用 Bakery。之后 Bakery 将你的服务打包成一个 AMI。使用一个如 Bakery 的 SaaS 产品意味着你不必浪费宝贵的时间来设置 AMI 创建基础架构。

每个虚拟机一个服务实例模式有许多优点。VM 的主要优点是每个服务实例运行是完全隔离的。它有固定数量的 CPU 和内存，而且不能从其他服务窃取资源。

将微服务部署为虚拟机的另一个优点是可以利用成熟的云基础架构。如 AWS 之类的云提供了很多有用的功能，比如负载平衡和自动扩缩。

将服务部署为虚拟机的另一个好处是它封装了服务的实现技术。一旦服务被打包成一个虚拟机，它就成为一个黑匣子。VM 的管理 API 成为部署服务的 API。部署变得更加简单、可靠。

然而，每个虚拟机一个服务实例模式也有一些缺点。一个缺点是资源利用率较低。每个服务实例都有一整个 VM 开销，包括操作系统部分。此外，在一个典型的公共 IaaS 中，VM 有着固定大小，并且可能未被充分利用。

此外，公共 IaaS 中的 VM 通常是收费的，无论它们是处于繁忙还是空闲状态。如 AWS 之类的 IaaS 虽然提供了自动扩缩功能，但[很难快速响应需求变化](http://techblog.netflix.com/2013/11/scryer-netflixs-predictive-auto-scaling.html)。因此，你经常需要过度配置 VM，从而增加部署成本。

这种方法的另一缺点是部署新版本的服务时很慢。由于大小原因，通常 VM 镜像构建很慢。此外，VM 实例化也会很慢，同样是因为大小原因。而且，操作系统也需要一些时间来启动。但请注意，这并不常见，因为已经存在由 Boxfuse 构建的轻量级 VM。

每个虚拟机一个服务实例模式的另一个缺点是你（或组织中的其他人）要对很多未划分的重担负责。除非你使用 Boxfuse 这类的工具来处理构建和管理虚拟机的开销，否则这将是你的责任。这个必要而又耗时的活动会分散你的核心业务。

接下来让我们看看另一种更轻量级的微服务部署方式，它也有许多与虚拟机一样的优势。

<a id="service-instance-per-container-pattern"></a>

### 6.3.2、每个容器一个服务实例模式

当你使用[每个容器一个服务实例模式](http://microservices.io/patterns/deployment/service-per-container.html)（Service Instance per Container）模式时，每个服务实例都在自己的容器中运行。容器是一个[操作系统级虚拟化机制](https://en.wikipedia.org/wiki/Operating-system-level_virtualization)。一个容器是由一个或多个运行在沙箱中的进程组成。从进程的角度来看，它们有自己的端口命名空间和根文件系统。你可以限制容器的内存和 CPU 资源。一些容器实现也有 I/O 速率限制。容器技术的相关例子有 [Docker](https://www.docker.com/) 和 [Solaris Zones](https://en.wikipedia.org/wiki/Solaris_Containers)。

图 6-3 展示了该模式的结构：

![图 6-3、服务可以运行在各自的容器中](resources/6-3.png)

要使用此模式，请将你的服务打包成一个容器镜像。容器镜像是由运行服务所需的应用和库组成的文件系统镜像。一些容器镜像由完整的 Linux 根文件系统组成。其它的则更加轻便。例如，要部署一个 Java 服务，你可以构建一个包含了 Java 运行时的容器镜像，有时也可能包含一个 Apache Tomcat 服务器和编译好的 Java 应用。

将服务打包成一个容器镜像后，你将启动一个或多个容器。通常在每台（个）物理或虚拟主机上运行多个容器。你可以使用集群管理工具（如 [Kubernetes](https://kubernetes.io/) 或 [Marathon](https://github.com/mesosphere/marathon)）来管理容器。集群管理工具将主机视为一个资源池。它根据容器所需的资源和每个主机上可用的资源来决定每个容器放置的位置。

每个容器一个服务实例模式模式有优点也有缺点。容器的优点与虚拟机的类似。它们将服务实例彼此隔离。你可以轻松地监控每个容器所消耗的资源。此外，与 VM 一样，容器封装了服务的实现技术。可将容器管理 API 作为服务管理 API。

然而，与虚拟机不同，容器是轻量级技术。容器镜像可以非常快速地构建。例如，在我的笔记本电脑上，将一个 [Spring Boot](http://projects.spring.io/spring-boot/) 应用打包成一个 Docker 容器只需要 5 秒钟的时间。容器也可以很快地启动，因为没有繁琐的操作系统引导机制。当一个容器启动时，它所运行的就是服务。

使用容器有一些缺点。虽然容器基础架构正在快速发展走向成熟，但它并不像虚拟机的基础架构那么成熟。此外，容器不像 VM 那样安全，因为容器彼此共享着主机的 OS 内核。

容器的另一个缺点是你需要负责管理这些庞大且无法区分的容器镜像。此外，除非你使用了托管容器解决方案［如 [Google Container Engine](https://cloud.google.com/container-engine/) 或 [Amazon EC2 Container Service](https://cloud.google.com/container-engine/)（ECS）］，否则你必须自己管理容器基础设施，可能还有运行容器的 VM 基础架构。

此外，容器通常部署在一个按单个 VM 收费的基础设施上。因此，如之前所述，当有处理负载峰值时，可能会产生超额配置 VM 的额外成本。

有趣的是，容器和 VM 之间的区别可能会有些模糊。如之前所述，Boxfuse VM 可以很快地构建和启动。[Clear Containers](https://clearlinux.org/features/clear-containers) 项目旨在创建轻量级分的虚拟机。[Unikernels](https://en.wikipedia.org/wiki/Unikernel) 也正在蓬勃发展。Docker 公司于 2016 年初收购了 Unikernel 系统。

还有一个日益流行的 server-less（无服务器）部署概念，它避免了「在容器中还是在虚拟机中部署服务」这一问题。

<a id="serverless-deployment"></a>

## 6.4、Serverless 部署

[AWS Lambda](https://aws.amazon.com/lambda/) 就是一个 serverless 部署技术示例。它支持 Java、Node.js 和 Python 服务。要部署微服务时，需将服务打包成 ZIP 文件并将上传到 AWS Lambda。你还要提供元数据，其中包括了被调用用来处理请求（又称为事件）的函数的名称。AWS Lambda 会自动运行足够的微服务服务实例来处理请求。你只需根据每个请求所用时间和内存消耗来付费。当然，问题往往出现在细节上，你很快注意到了 AWS Lambda 的局限性。但作为开发人员的你或组织中的任何人都无需担心服务器、虚拟机或容器方面的任何问题 ，这非常有吸引力，足以令人难以置信。

Lambda 函数是无状态服务。它通常通过调用 AWS 服务来处理请求。例如，当图片上传到 S3 存储桶时 Lambda 函数将被调用，该函数可插入一条记录到 DynamoDB 图片表中，并将消息发布到 Kinesis 流以触发图片处理。Lambda 函数还可以调用第三方 Web 服务。

有四种方法调用 Lambda 函数：

- 直接使用 Web 服务请求
- 自动响应一个 AWS 服务（如 S3、DynamoDB、Kinesis 或 Simple Email Service）生成的事件
- 通过 AWS API 网关自动处理来自应用客户端的 HTTP 请求
- 按照一个类似 cron 的时间表，定期执行

正如你所见，AWS Lambda 是一个便捷的微服务部署方式。基于请求的定价意味着你只需为服务实际执行的工作付费。另外，由于你不需要对 IT 基础架构负任何责任，因此可以专注于开发应用。

然而，其也存在一些明显的局限性。Lambda 函数不适用于部署长时间运行的服务，例如消耗第三方消息代理消息的服务。请求必须在 300 秒内完成。服务必须是无状态的，因为理论上，AWS Lambda 可能为每个请求运行一个单独的实例。它们必须使用受支持的语言来编写。服务启动速度也必须快，否则，它们可能会因超时而终止。

<a id="summary"></a>

## 6.5、总结

部署微服务应用充满着挑战。你可能有数个甚至数百个使用了各种语言和框架编写的服务。每个应用都是一个迷你应用，有自己特定的部署、资源、扩展和监控需求。有几个微服务部署模式，包括每个虚拟机一个服务实例和每个容器一个服务实例模式。部署微服务的另一个有趣的选择是 AWS Lambda，一种 serverless 方式。在本书的下一章也是最后一章中，我们将介绍如何将单体应用迁移到微服务架构。

<a id="microservices-in-action"></a>

## 微服务实战：使用 NGINX 在不同主机上部署微服务

by Floyd Smith

NGINX 对于各种类型的部署有许多优势 —— 无论是单体应用、微服务应用还是混合应用（将在下一章介绍）。使用 NGINX，你可以智能抽取不同的部署环境出来并整合入 NGINX。如果你使用针对不同部署环境的工具，则有许多应用功能将以不同的方式工作，但如果使用 NGINX，那么在所有环境中都可以使用相同的方式进行工作。

这一特性也为 NGINX 和 NGINX Plus 带来了第二个优势：通过在多个部署环境中同时运行应用来扩展应用的能力。假设你拥有和管理着的本地服务器，但是你的应用使用量正在增长，并预计将超出这些服务器可以处理的峰值。如果你已经使用了 NGINX，你就有了一个强大的选择：扩展到云端 —— 例如，[扩展到 AWS 上](https://www.nginx.com/products/nginx-plus-aws/)，而不是购买、配置和保持额外的服务器来为了以防万一。也就是说，当你的本地服务器上的流量达到容量限制时，可根据需要在云中启动其他微服务实例来处理。

这只是因使用 NGINX 变得更加灵活的一个例子。

[NGINX 微服务参考架构](https://www.nginx.com/blog/introducing-the-nginx-microservices-reference-architecture/)被明确设计为支持这种灵活部署方式，其假设在开发和部署期间使用容器技术。如果你还没尝试，可以考虑转移到容器、NGINX 或 NGINX Plus，以轻松地转移到微服务，使你的应用、开发和部署灵活性以及人员更具前瞻性。
