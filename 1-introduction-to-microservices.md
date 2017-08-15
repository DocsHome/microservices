# 1、微服务简介
目前微服务受到很多关注：文章、博客、社交媒体上的讨论和会议演讲。他们正在迅速走向[加德纳技术成熟度曲线（Gartner Hype cycle）](http://www.gartner.com/technology/research/methodologies/hype-cycle.jsp)的高峰。与此同时，也有持怀疑态度的软件社区人员认为微服务没什么新鲜可言。反对者声称它的思想只是面向服务架构（SOA）的重塑。然而，无论是炒作还是怀疑，不可否认微服务架构模式具有非常明显的优势——特别是在实施敏捷开发和复杂的企业应用交付方面。

本章节是七章之中介绍关于的设计、构建和部署微服务方面的内容。您将了解微服务的由来和与传统[单体应用模式](http://microservices.io/patterns/monolithic.html)的比较。这本电子书描述了许多关于微服务架构方面的内容。无论是对您的项目的意义，或者是如何去应用它，您都将了解到微服务架构模式的优点和缺点。

我们先来看看为什么要考虑使用微服务。

## 1.1、构建单体应用
我们想一下，您开始制作一个新的打车应用，打算与 Uber 和 Hailo 竞争。经过初步交流与需求收集，您将手动或者使用类似 Rails、Spring Boot、Play或者Maven等平台来生成一个新项目。

这个新应用是一个模块化的六边形架构，如图 1-1 所示：

![图 1-1、一个简单的打车应用](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/resources/1-1.png)

