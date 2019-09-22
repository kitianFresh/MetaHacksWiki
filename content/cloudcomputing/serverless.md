---
title: "serverless"
date: 2019-09-01 16:20
---

# 什么是 Serverless
和软件领域很多新趋势一样，Serverless 目前还没有一个清晰的定义。对于初学者而言，可以认为它包含两个不同但是又有重合的领域：
1. Serverless 首先用来描述那些 完全集成第三方云上托管的服务 来完成服务端代码逻辑和管理状态 的应用。这些应用通常被称为 “富客户端” 应用，比如 单页WebAPP，移动App，他们都是通过直接调用庞大的云上生态体系服务来完成开发，像数据库（比如Parse，Firebase），认证服务（比如Auth0, AWS Cognito）等等。这些类型的服务再过去一直被描述为 “(Mobile) Backend as a Service”, 在文章后续部分我会使用 BaaS 来指代它。
2. Serverless 也可以指代这样一类应用，它们仍然由应用开发者编写服务端逻辑，但不同于传统架构运行并且托管在服务器（物理机虚拟机）上，而是运行在无状态的容器中，这些容器实例通过事件触发而短暂运行（比如仅仅是一次函数调用，运行完毕以后就会销毁），并且通常完全由第三方管理，使用者无需关系这些容器的生命周期以及资源情况。这就是所谓的 “Function as a Service” 或者说 “FaaS”

在这篇文章中，我们主要聚焦于 FaaS。 不仅仅是因为Serverless更新并且有更多宣传和炒作，更是因为它对于我们思考技术架构有巨大差异。

## 一组小例子
### UI 驱动的应用

### 消息驱动的应用

## 解构 “Function as a Service”

### 状态（State）


### 执行时间（Execution duration）

### 启动延迟和冷启动

### 工具


### 开源

## 什么不是 Serverless

### Serverless 和 PaaS

### Serverless 和 容器

### Serverless 和 NoOps（无运维）

### Serverless 和 存储过程即服务（Stored Procedures as a Service）


# 优点

# 缺点

# 总结