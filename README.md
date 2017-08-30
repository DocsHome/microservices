# 《微服务：从设计到部署》中文版
本仓库为电子书 [Designing and Deploying Microservices](https://www.nginx.com/resources/library/designing-deploying-microservices/) 的中文翻译版本，不涉及任何商业利益，纯属个人爱好。

![Cover](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/resources/cover.png)

## 目录

### [0、前言](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/0-foreword.md)

### [1、微服务简介](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/1-introduction-to-microservices.md#1微服务简介)
- [1.1、构建单体应用](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/1-introduction-to-microservices.md#11构建单体应用)
- [1.2、走向单体地狱](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/1-introduction-to-microservices.md#12走向单体地狱)
- [1.3、微服务-解决复杂问题](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/1-introduction-to-microservices.md#13微服务-解决复杂问题)
- [1.4、微服务的优点](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/1-introduction-to-microservices.md#14微服务的优点)
- [1.5、微服务的缺点](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/1-introduction-to-microservices.md#15微服务的缺点)
- [1.6、总结](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/1-introduction-to-microservices.md#16总结)
- [微服务实战：NGINX Plus 作为反向代理服务器](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/1-introduction-to-microservices.md#微服务实战nginx-plus-作为反向代理服务器)

### [2、使用API网关](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/2-using-an-api-gateway.md)
- [2.1、简介](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/2-using-an-api-gateway.md#21简介)
- [2.2、客户端与微服务直接通信](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/2-using-an-api-gateway.md#22客户端与微服务直接通信)
- [2.3、使用 API 网关](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/2-using-an-api-gateway.md#23使用API网关)
- [2.4、API 网关的优点和缺点](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/2-using-an-api-gateway.md#24API网关的优点和缺点)
- [2.5、实施 API 网关](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/2-using-an-api-gateway.md#25实施API网关)
    - [2.5.1、性能与扩展](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/2-using-an-api-gateway.md#251性能与扩展)
    - [2.5.2、使用响应式编程模型](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/2-using-an-api-gateway.md#252使用响应式编程模型)
    - [2.5.3、服务调用](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/2-using-an-api-gateway.md#253服务调用)
    - [2.5.4、服务发现](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/2-using-an-api-gateway.md#254服务发现)
    - [2.5.5、处理部分故障](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/2-using-an-api-gateway.md#255处理部分故障)
- [2.6、总结](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/2-using-an-api-gateway.md#26总结)
- [微服务实战：NGINX Plus 作为 API 网关](https://github.com/oopsguy/microservices-from-design-to-deployment-chinese/blob/master/2-using-an-api-gateway.md#微服务实战nginx-plus-作为api-网关)

### 3、进程间通信
- 3.1、简介
- 3.2、交互风格
- 3.3、定义 API
- 3.4、演变 API
- 3.5、处理部分失败
- 3.6、IPC 技术
- 3.7、异步，基于消息通信
- 3.8、同步，请求/响应 IPC
- 3.9、REST
- 3.10、Thrift
- 3.11、消息格式
- 3.12、总结
- 微服务实战：NGINX 和应用架构

### 4、服务发现
- 4.1、为什么使用服务发现
- 4.2、客户端发现模式
- 4.3、服务端发现模式
- 4.4、服务注册表
- 4.5、服务注册选项
- 4.6、自注册模式
- 4.7、第三方注册模式
- 4.8、总结
- 微服务实战：NGINX 的灵活性

### 5、微服务的事件驱动数据管理
- 5.1、微服务与分布式数据管理问题
- 5.2、事件驱动架构
- 5.3、实现原子性
- 5.4、使用本地事务发布事件
- 5.5、挖掘数据库事务日志
- 5.6、使用事件溯源
- 5.7、总结
- 微服务实战：NGINX 与存储优化

### 6、选择微服务部署策略
- 6.1、动机
- 6.2、单主机多服务实例模式
- 6.3、单主机服务实例模式
- 6.4、单虚拟机服务实例模式
- 6.5、单容器服务实例模式
- 6.6、无服务部署
- 6.7、总结
- 微服务实战：部署微服务

### 7、用NGINX跨越不同的主机
- 7.1、重构单体为微服务
- 7.2、微服务重构概述
- 7.3、策略1：停止挖掘
- 7.4、策略2：分离前后端
- 7.5、策略3：提取服务
- 7.6、优先将哪些模块转为微服务
- 7.7、怎样提取模块
- 7.8、总结
- 7.9、微服务实战：用 NGINX 驯服单体应用
- 微服务与 NGINX 的相关资源

## Licenses
![知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议](https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png)

本作品采用[知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。
