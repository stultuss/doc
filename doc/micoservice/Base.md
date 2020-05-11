
微服务基础
=========================

> 本文内容主要简单描述微服务架构相关技术栈，部分内容来自阮一峰的《软件架构入门》

## 一. 前言

微服务的概念来源于 Martin Fowler 在2014年发表的名为《[Microservices](http://martinfowler.com/articles/microservices.html)》的文章，文章中正式提出了微服务架构的风格，之后由 Netflix 经过多年大规模生产验证，并落地了一整套的微服务基础组件。由于 Netflix 的成功经验，微服务架构逐渐开始在业界流行，之后 Privotal 公司将 Netflix 开源的微服务组件集成到了 Spring 体系中，推出了 Spring Cloud 微服务开发技术栈。

近年来，由于 Docker，PaaS，gRPC 等新技术崭露头角，微服务技术生态也在不断变化，每个人都在谈微服务，每个人都说自己的系统是基于微服务的。那么什么是微服务？

## 二. 什么是微服务

微服务架构是 SOA 架构（Service-Oriented Architecture）的升级，微服务架构是将 SOA 架构中的单个业务系统拆分成多个可独立开发，设计，运行以及可独立部署的单元，这些单元之间互相解耦，并通过远程通信协议进行联系，从而实现之前 SOA 架构中单个业务系统的功能。

从Martin Fowler 的文章中可以得出，微服务具有以下特征：

1. 服务组件化 / Componentization via Services

   - 可独立替换和升级
   - 可独立部署
   - 单一职责原则原则

2. 围绕业务组织能力/ Organized around Business Capabilities

   - 用《康威定律》可以进行概括，简单来说就是 "组织决定架构"
   - 理解不是很透彻，个人理解是将业务按团队拆分，或反之

3. 是产品不是项目 / Products not Projects

   - 一个好的架构通常都是演化出来的，而非完成即交付。

4. 智能端点和哑管道 / Smart endpoints and dumb pipes：

   - 完全不理解含义，引用微服务社区的主张，基于微服务构建的应用程序的目标是**尽可能的解耦和尽可能的内聚** 。例如使用简单的REST风格的协议来编排

5. 去中心化治理 / Decentralized Governance

   - 将一个整体业务拆分成多个服务，每个服务都有可以有各自独特的构建方式。

6. 去中心化数据管理 / Decentralized Data Management

   - 需要理解三个概念
     - 领域驱动设计（DDD）
     - 边界上下文（Bounded Context）
     - 职责分离模式（CQRS）
   - 另外微服务架构强调服务间的无事务协作，关注最终一致性和事后补偿，作为架构设计师，则需要对此进行权衡：强一致性下的业务损失的代价是否小于错误修复的代价？

7. 基础设施自动化 / Infrastructure Automation

   - 自动化测试
   - 自动化部署
   - 服务编排和治理

8. 为失效设计 / Design for Failure、

   - 将微服务架构设计成**能够容忍服务失效**。任何服务调用都可能因为服务提供者不可用而失败，客户端必须尽可能优雅的应对这种失败。
   - 有足够的方案应对服务失效:
     - 服务监控
     - 服务重启
     - 健康检查
     - 告警机制
     - 断路器
     - 服务熔断
     - 事件回溯
     - 等等

9. 进化式设计 / Evolutionary Design

   - **架构是通过演化而变得更加完善**
   - **服务可独立的更换和升级**

### 微服务架构的优点

通过微服务的特征，我们可以了解到微服务的优点很明显：

- **扩展性好**：各服务之间低耦合，单一服务业务简单，只需要关注当前业务。
- **容易部署**：软件从单一可部署单元，拆成了多个服务，每个服务也都是可部署单元。
- **局部修改**：局部更新，每个单元都可以进行持续集成式的开发，可以做到实时部署，不间断升级。
- **语言无关**：每一个单元都可以通过最合适的编程语言进行开发。

### 微服务架构的缺点

一切看起来真的很美好，但你的了解微服务了吗？

微服务架构从来就不是解决软件工程的银弹，首先来看一张图，再问问自己准备好挑战微服务架构了没有？图片来源《[阿里巴巴之微服务之熵](https://zhuanlan.zhihu.com/p/31324360)》

![微服务之殇](https://raw.githubusercontent.com/niklaus0823/niklaus0823.github.io/master/_posts/images/2018-03-27-01.jpg)

从这张图上我们可以看出，伴随着业务增长，熵值成指数级别上升，怎么去梳理和解决这个问题就尤为重要。

以下我列举了微服务的几个缺点，目前已经有思路去解决这些问题：

- **依赖关系复杂** ：一个业务系统可能会拆分的很细，单元与单元之间的依赖关系会变得非常复杂，当系统越来越庞大，如何去管理这些依赖关系，是一个很重要的工作。
- **故障定位困难** ：由于依赖关系复杂，导致微服务之间的互相调用关系，变得凌乱而笨重，如果整个调用链非常长的情况下（同步），性能会受到影响，问题排查也会非常困难
- **事务操作困难** ：分布式的本质使得微服务架构很难实现原子性操作，事务回滚会变得非常困难。
- **热点/瓶颈** ：错误的微服务设计中容易将某个微服务成为热点，造成整体性能的瓶颈，如何排查，解决是一个课题需要研究。
- **过度设计** ：正确的设计微服务的粒度是一个很难的工作，避免为了“微服务”而“微服务”，理解DDD和CQRS是非常重要的。

再泼一盆冷水：《[八种可以让微服务失效的方法](http://www.russmiles.com/essais/8-ways-to-lose-at-microservices-adoption)》

### 其他思考：

- **远程调用比进程内调用更昂贵** ：微服务间大量的远程调用的资源浪费。 
- **API设计粒度过粗，不方便使用** ：是否需要采用 [GraphQL](https://graphql.cn/) 让接口更灵活？？？

## 三. 基础架构与技术栈

>  本段落是笔者以 nodejs 开发者的视角，整理的技术栈列表，之后会对每一个内容进行介绍和实践。

谈完了什么是微服务，下面列举我已经完成的和未来将要进行的实践与调研。

### 1. 基础

- [ ] 数据存储格式
- [ ] 数据通信框架
- [ ] 接口设计规范

### 2. 部署

- [ ] 持续集成
- [ ] 镜像治理
- [ ] 容器编排和调度
- [ ] 服务发布机制（蓝绿，金丝雀，灰度）

### 3. 安全

- [ ] 单点登录

### 4. 组件

- [ ] 消息服务
- [ ] 缓存服务
- [ ] 数据库中间件 + 数据库

### 5. 监控

- [ ] 运行监控
- [ ] 链路监控
- [ ] Metrics 监控

### 6. 支撑 

- [ ] 服务注册与发现
- [ ] 负载均衡 
- [ ] 配置中心

### 7. 其他

- [ ] 接口幂等性设计
- [ ] 服务容错（超时，熔断，隔离，限流，降级）
- [ ] 分布式一致性（CAP定理，两阶段提交，最终一致性，补偿事务）

## 四. 题外话

微服务概念的发明者Martin Fowler，公开说[“个子矮，就不要用微服务”](https://martinfowler.com/bliki/MicroservicePrerequisites.html)

一个团队必须拥有三种能力，才能考虑使用微服务：

- 服务器的快速扩容
- 监控
- 应用的快速部署

换句话说，devops 能力是微服务的前提。