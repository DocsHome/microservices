# 1、微服务简介
目前微服务受到很多关注：文章、博客、社交媒体上的讨论和会议演讲。他们正在迅速走向[加德纳技术成熟度曲线（Gartner Hype cycle）](http://www.gartner.com/technology/research/methodologies/hype-cycle.jsp)的高峰。与此同时，也有持怀疑态度的软件社区人员认为微服务没什么新鲜可言。反对者声称它的思想只是面向服务架构（SOA）的重塑。然而，无论是炒作还是怀疑，不可否认微服务架构模式具有非常明显的优势——特别是在实施敏捷开发和复杂的企业应用交付方面。

本章节是七章之中介绍关于的设计、构建和部署微服务方面的内容。您将了解微服务的由来和与传统[单体应用模式](http://microservices.io/patterns/monolithic.html)的比较。这本电子书描述了许多关于微服务架构方面的内容。无论是对您的项目的意义，或者是如何去应用它，您都将了解到微服务架构模式的优点和缺点。

我们先来看看为什么要考虑使用微服务。

## 1.1、构建单体应用
我们想一下，您开始制作一个新的打车应用，打算与 Uber 和 Hailo 竞争。经过初步交流与需求收集，您将手动或者使用类似 Rails、Spring Boot、Play或者Maven等平台来生成一个新项目。

这个新应用是一个模块化的六边形架构，如图 1-1 所示：

![图 1-1、一个简单的打车应用](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/resources/1-1.png)

应用程序的核心是由模块实现的业务逻辑，它定义了服务、领域对象和事件。围绕核心的是与外部世界接口对接的适配器。适配器示例包括了数据库访问组件、生产和消费消息的消息组件和 web 组件，它们暴露了 API 或者实现了一个 UI。

尽管有一个逻辑模块化的架构，但应用程序被作为一个整体进行打包和部署。实际格式取决于应用程序的语言和框架。例如，许多 Java 应用程序被打包成 WAR 文件并部署在如 Tomcat 或者 Jetty 之类的应用服务器上。其他 Java 应用程序是打包成自包含（self-contained）的可执行 JAR。类似地，Rails 和 Node.js 应用程序被打包为目录层次结构。

以这种风格编写的应用是很常见的。他们很容易开发，因为我们的 IDE 和其他工具专注于构建单个应用程序。这些应用程序也很容易测试。您可以通过简单地启动并使用如 Selenium 测试包来测试 UI 以轻松地实现端到端（end-to-end）测试。单体应用同样易于部署。你只需拷贝打包好的应用程序到服务器。您还可以通过运行多个副本和结合负载均衡器来扩展应用。在项目的早期阶段，它运作良好。

## 1.2、走向单体地狱
不幸的是，这种简单的方法有很大的局限性。成功的应用有一个趋势，随着时间推移而变得越来越臃肿。在每个冲刺时期，您的开发团队实现了更多的用户需求，这意味着添加了许多行代码。几年之后，您的小而简单的应用将会逐渐成长为[庞然大物似的单体](http://microservices.io/patterns/monolithic.html)。为了给出一个极端的示例，我最近和一位开发者做了交谈，他正在编写一个工具用于从他们的数百万行代码（lines of code，LOC）应用中分析数千个 JAR 之间的依赖。我相信这是大量开发者在多年齐心协力下创造了这样的野兽。

一旦您的应用程序成为了一个庞大、复杂的单体，您的开发组织可能陷入了一个令人痛苦的世界。敏捷开发和交付的任何一次尝试都将原地徘徊。一个主要问题是应用程序实在是非常复杂。对于任何一个开发人员来说，这是在太大了，这是可以理解的。最终，正确地修复 bug 和实现新功能变得非常困难且耗时。此外，这个趋势就像是往下的螺旋。如果基本代码难以理解，那么改变将不会变得正确。您最终得到的是一个巨大且不可思议的[大泥球](http://www.laputan.org/mud/)。

应用程序的规模也将减缓发展。应用程序越大，启动的时间越长。我[调查](https://plainoldobjects.com/2015/05/13/monstrous-monoliths-how-bad-can-it-get/)过开发者们的单体应用的大小和性能，一些报告的启动时间为 12 分钟。我也听说过应用程序启动需要 40 分钟以上的怪事。如果开发人员经常要重启应用服务器，那么很大一部分时间都是在等待中度过，他们的生产力将受到限制。

另一个大问题是，复杂的单体应用本身就是持续部署的障碍。如今，SaaS 应用发展到了每天可将变更推送到生产中多次。这对于复杂的单体非常困难，因为您必须重新部署整个应用程序才能更新其中任何一部分。这对我之前提到的漫长的启动时间也不会有什么帮助。此外，因为变化的影响通常不是很清楚，您很可能需要做大量的手工测试。因此，持续部署是不可能做到的。

当不同模块存在资源需求冲突时，单体应用也可能难以扩展。例如，一个模块可能会执行 CPU 密集型图像处理逻辑，理想情况下是部署在 [Amazon EC2 Compute Optimized 实例](http://aws.amazon.com/about-aws/whats-new/2013/11/14/announcing-new-amazon-ec2-compute-optimized-instances/)中。另一个模块可能是内存数据库，最适合 [EC2 Memory-optimized 实例](https://aws.amazon.com/cn/about-aws/whats-new/2014/04/10/r3-announcing-the-next-generation-of-amazon-ec2-memory-optimized-instances/)。然而，因为这些模块被部署在一起，您必须在硬件选择上做出妥协。

单体应用的另一个问题是可靠性。因为所有模块运行在同一进程。任何模块的一个 bug，比如内存泄漏，可能会拖垮整个进程。此外，由于应用程序的所有的实例都是相同的，该错误将影响到整个应用的可用性。

最后但并非是最不重要的，单体应用使得采用新的框架和语言变得非常困难。例如，我们假设您有 200 万行代码使用了 XYZ 框架编写。使用较新的 ABC 框架来重写整个应用，这将是非常昂贵的（在时间和成本方面），即使那个框架非常好。因此，这对于采用新技术是一个非常大的障碍。您在项目开始时无论选择何种新技术都将会感到困扰。

总结一下：您有一个成功的关键业务应用程序，它已经发展成为一个只有少数开发人员（如果有的话）能够理解的巨大单体。它使用了过时、非生产性技术编写的，这使得招聘优秀开发人员变得困难。应用程序难以扩展，不可靠。因此敏捷开发和应用交付是不可能的。

那么您能做些什么呢？

## 1.3、微服务-解决复杂问题
许多组织，如 Amazon、eBay 和 [Netflix](http://www.nginx.com/blog/microservices-at-netflix-architectural-best-practices/)，已经采用现在所谓的[微服务架构模式](http://microservices.io/patterns/microservices.html)解决了这个问题，而不是构建一个臃肿的单体应用。它的思路是将您的应用程序分解成一套较小的互连服务。

一个服务通常实现了一组不同的特性或者功能，例如订单管理、客户管理等。每一个微服务都是一个迷你应用，它自己的六边形架构包括业务逻辑以及多个适配器。

一些微服务会暴露一个供其他微服务或应用客户端消费的 API。其他微服务可能实现了一个 web UI。在运行时，每个实例通常是一个云虚拟机（virtual machine，VM）或者一个 Docker 容器。

例如，前面描述的系统可能分解成如图 1-2 所示：

![图 1-2、一个单体应用分解成微服务](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/resources/1-2.png)

应用程序的每个功能区域现在都由自己的微服务实现。此外，Web 应用程序被分为一组更简单的 Web 应用程序。例如，以我们的出粗车为例，一个专门是乘客的，一个专门是司机的。这使得它更容易地为特定的用户、司机、设备或者专门的用例部署不同的场景。每个后端服务暴露一个 REST API，大部分的服务消费由其他服务提供的 API。例如，司机管理使用了通知服务器来告知一个可用司机关于一个潜在路程。UI 服务调用了其他服务来渲染页面。服务也可以使用异步、基于消息的通信。本电子书后面将会详细介绍服务间的通信。

一些 REST API 也暴露给移动端应用供司机和乘客使用。然而，应用不能直接访问后端服务。相反，他们之间的通信是由一个称为 [API 网关](http://microservices.io/patterns/apigateway.html)（API Cateway）的中介负责。API 网关负责负载均衡、缓存、访问控制、API计量和监控，[可以通过使用 NGINX 来实现](http://www.nginx.com/solutions/api-gateway/)。第 2 章详细讨论 API 网关。

![图 1-3、开发和交付中的缩放立方（Scale Cube）](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/resources/1-3.png)

微服务架构模式相当于此缩放立方的 Y 轴坐标，此立方是一个来自[《架构即未来》](http://theartofscalability.com/)的三维伸缩模型。另外两个坐标轴是由运行多个相同应用程序副本的负载均衡器组成的 X 轴缩放和 Z 轴坐标（或者数据分区），其中请求的属性（例如，一行记录的主键或者客户标识）用于将请求路由到特定的服务器。

应用程序通常将这三种类型的坐标方式一起使用。Y 轴坐标将应用分解成微服务，如图 1-2 所示。

在运行时，X 坐标轴上运行着服务的多个实例，每一个服务配合负载均衡器以满足吞吐量和可用性。某些应用程序也有可能使用 Z 坐标轴来进行分区服务。图 1-4 展示了如何用 Docker 将旅途管理（Trip Management）服务部署到 Amazon EC2 上运行。

![图 1-4、使用 Docker 部署旅途管理服务](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/resources/1-4.png)

在运行时，旅途管理服务由多个服务实例组成，每个服务实例是一个 Docker 容器。为了实现高可用，容器是在多个云虚拟机上运行的。服务实例的前方是一个[如 NGINX 的负载均衡器](http://www.nginx.com/solutions/load-balancing/)，用于跨实例分发请求。负载均衡器也可以处理其他问题，如[缓存](http://www.nginx.com/resources/admin-guide/content-caching/)、[访问控制](http://www.nginx.com/resources/admin-guide/restricting-access/)、[API 度量](http://www.nginx.com/solutions/api-gateway/)和[监控](http://www.nginx.com/products/live-activity-monitoring/)。

微服务架构模式明显影响到了应用程序与数据库之间的关系。与与其他共享单个数据库模式（schema）服务不同，每一个服务都有自己的数据库模式。一方面，这种做法是与企业级数据库数据模型的想法不符，此外，它经常导致一些数据冗余。然而，如果您想从微服务中受益，每一个服务都应该有自己的数据库模式。因为它做到了松耦合。图 1-5 展示了数据库架构示例应用程序。

每个服务都有自己的数据库。而且，服务可以使用一种最适合其需求、号称多语言持久架构（polyglot persistence architecture）的数据库。例如，司机管理，找到与潜在乘客接近的司机必须使用支持高效地理查询的数据库。

![图 1-5、打车应用的数据库架构](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/resources/1-4.png)

从表面上看，微服务架构模式类似于 SOA。微服务是由一组服务组成。然而，换另一种方式去思考微服务架构模式，它是没有商业化的 SOA，没有集成 [Web 服务规范](http://en.wikipedia.org/wiki/List_of_web_service_specifications)（WS-\*）和企业服务总线（Enterprise Service Bus，ESB）。基于微服务的应用支持更简单、轻量级的协议，例如，REST，而不是 WS-\*。他们也尽量避免使用 ESB，而是实现微服务本身具有类似 ESB 的功能。微服务架构也拒绝了 SOA 的其他部分，例如，数据访问[规范模式](https://en.wikipedia.org/wiki/Canonical_schema_pattern)概念。

## 1.4、微服务的优点
**待续……**