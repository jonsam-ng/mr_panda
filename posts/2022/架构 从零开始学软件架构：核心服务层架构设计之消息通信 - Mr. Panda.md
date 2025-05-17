---
date: 2022-01-01
title: 从零开始学软件架构：核心服务层架构设计之消息通信 - Mr. Panda
# category: 
# tags: 
# description:
---

# [架构] 从零开始学软件架构：核心服务层架构设计之消息通信 - Mr. Panda

---
本文是从零开始学软件架构系列文章的第四篇，主题为核心服务层架构设计。本文的主要内容包括：微服务架构、dubbo 服务治理、异步化消息服务等内容。本文的学习目标是通过传统服务架构、SOA 架构、微服务架构的演进过程，了解微服务的架构理念；学习 Dubbo 服务治理的功能和特性、Dubbo 的组件角色和基本原理；学习异步化消息服务 JMS、Kafka、RocketMQ 三种技术，理解 Kafka 和 RocketMQ 的基本原理。

## 微服务

背景问题：

-   什么是服务？
-   什么是微服务？
-   为什么要有微服务？

### 传统服务架构

传统的服务和服务分层：Web 层、业务层、数据访问层、数据持久层。

传统的整体式架构都是模块化的设计逻辑，如展示（Views）、应用程序逻辑（Controller）、业务逻辑（Service）和数据访问对象（Dao），程序在编写完成后被打包部署为一个具体的应用。如果要对系统进行水平扩展，通常情况下，只需要增加服务器的数量，并将打包好的应用拷贝到不同的服务器，然后通过负载均衡器（Nginx）就可以轻松实现应用的水平扩展。

传统服务的缺点：

-   所有服务耦合在一起
-   隔离性弱，互相影响
-   部署臃肿
-   开发维护困难

### SOA 架构

SOA：**面向服务的架构（Service-Oriented Architecture）**，思路是把应用中相近的功能聚合在一起，以服务的形式提供出去。虽然SOA解决了整体式架构中的问题，但多数情况下，SOA中相互独立的服务仍然会部署在同一个运行环境中。和整体式架构类似，随着业务功能的增多，SOA的服务会变得越来越复杂。

### 微服务架构

服务化：

服务化就是把传统单体应用中通过 JAR 包依赖产生的本地方法调用，改造成 RPC 接口产生的远程方法调用。这些服务是围绕业务功能构建的，可以通过全自动部署机制独立部署。 这些服务的集中管理最少，可以用不同的编程语言编写，并使用不同的数据存储技术。

微服务：

微服务是指开发一个单个小型的但有业务功能的服务，每个服务都有自己的处理和轻量通讯机制，可以部署在单个或多个服务器上。

微服务架构风格是一种将单个应用程序作为一套小型服务开发的方法，每种应用程序都在自己的进程中运行，并与轻量级机制（通常是HTTP资源API）进行通信。得益于以 Docker 为代表的容器化技术的成熟以及 DevOps 文化的的兴起，服务化的思想进一步演化，演变成我们今天所熟知的微服务。  

优点：

-   服务高内聚，低耦合，服务拆分粒度更细：微服务可以说是更细维度的服务化，小到一个子模块，只要该模块依赖的资源与其他模块都没有关系，那么就可以拆分为一个微服务。
-   隔离性强，不会互相影响。
-   单独部署：传统的单体架构是以整个系统为单位进行部署，而微服务则是以每一个独立组件（例如用户服务，商品服务）为单位进行部署。
-   独立开发、维护，分工明确：每个微服务都可以交由一个小团队进行开发，测试维护部署，并对整个生命周期负责。

缺点：

-   开发人员必须处理创建分布式系统的复杂性。
-   部署的复杂性。
-   增加内存消耗。

微服务要解决的问题：

-   服务治理（服务调用通信，健康管理，**限流熔断**等）
-   数据一致性（分布式事务的数据一致性问题）
-   调用性能
-   研发流程，调试，部署

**微服务架构与SOA的区别**：

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648459111-image.jpeg)

微服务架构与SOA的区别

微服务架构的组件：

-   服务注册中心：注册系统中所有服务的地方。
-   服务注册：服务提供方将自己调用地址注册到服务注册中心，让服务调用方能够方便地找到自己。
-   服务发现：服务调用方从服务注册中心找到自己需要调用服务的地址。
-   负载均衡：服务提供方一般以多实例的形式提供服务，使用负载均衡能够让服务调用方连接到合适的服务节点。
-   服务容错：通过断路器（也称熔断器）等一系列的服务保护机制，保证服务调用者在调用异常服务时能快速地返回结果，避免大量的同步等待。
-   服务网关：也称为API网关，是服务调用的唯一入口，可以在这个组件中实现用户鉴权、动态路由、灰度发布、负载限流等功能。
-   分布式配置中心：将本地化的配置信息（properties、yml、yaml等）注册到配置中心，实现程序包在开发、测试、生产环境的无差别性，方便程序包的迁移。

微服务架构的技术选型（Java）：

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648442533-image.png)

微服务架构的技术选型（Java）

-   微服务实例的开发：SpringBoot
-   服务的注册与发现：Spring Cloud Eureka
-   负载均衡：Spring Cloud Ribbon
-   服务容错：Spring Cloud Hystrix
-   API网关：Spring Cloud Zuul
-   分布式配置中心：Spring Cloud Config
-   调试：Swagger
-   部署：Docker
-   持续集成：Jenkins

## Dubbo 服务治理

Dubbo 是用来解决微服务内服务治理问题的轻量级开源 **Java RPC 框架**。

其最大的特点是按照分层的方式来架构，使用这种方式可以使各个层之间解耦合（或者最大限度地松耦合）。从服务模型的角度来看，Dubbo 采用的是一种非常简单的模型，要么是提供方提供服务，要么是消费方消费服务，所以基于这一点可以抽象出服务提供方（Provider）和服务消费方（Consumer）两个角色。

三大核心能力：**面向接口的远程方法调用、智能容错和负载均衡、服务自动注册和发现**。

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648459098-image.png)

Dubbo 的服务治理

### 服务治理的功能和特性

dubbo 服务治理具有以下的功能：

-   服务提供者注册服务
-   服务消费者获取服务，并通过负载均衡策略选择服务提供者
-   动态增减服务提供者和服务消费者
-   服务监控
-   服务限流
-   服务降级
-   高容错
-   定制化开发

特性：

<table><tbody><tr><td><strong>特性</strong></td><td><strong>描述</strong></td></tr><tr><td>透明远程调用</td><td>就像调用本地方法一样调用远程方法；只需简单配置，没有任何 API 侵入</td></tr><tr><td>负载均衡机制</td><td>Client 端 LB，可在内网替代 F5 等硬件负载均衡器</td></tr><tr><td>容错重试机制</td><td>服务 Mock 数据，重试次数、超时机制等</td></tr><tr><td>自动注册发现</td><td>注册中心基于接口名查询服务提供者的 IP 地址，并且能够平滑添加或删除服务提供者</td></tr><tr><td>性能日志监控</td><td>Monitor 统计服务的调用次调和调用时间的监控中心</td></tr><tr><td>服务治理中心</td><td>路由规则，动态配置，服务降级，访问控制，权重调整，负载均衡，等手动配置</td></tr><tr><td>自动治理中心</td><td>熔断限流机制、自动权重调整等（因此可以搭配SpringCloud的熔断机制等进行开发）</td></tr></tbody></table>

dubbo 的特性

Dubbo 的核心功能：

-   Remoting：远程通讯，提供对多种 NIO 框架抽象封装，包括“同步转异步”和“请求-响应”模式的信息交换方式。
-   Cluster：服务框架，提供基于接口方法的透明远程过程调用，包括多协议支持，以及软负载均衡，失败容错，地址路由，动态配置等集群支持。
-   Registry：服务注册中心，服务自动发现: 基于注册中心目录服务，使服务消费方能动态的查找服务提供方，使地址透明，使服务提供方可以平滑增加或减少机器。

### Dubbo 的组件角色

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648459665-image.png)

Dubbo 组件结构图

组件角色说明：

-   Provider：暴露服务的服务提供方；
-   Consumer：调用远程服务的服务消费方；
-   Registry：服务注册与发现的注册中心；
-   Monitor：统计服务的调用次调和调用时间的监控中心；
-   Container：服务运行容器。

调用关系说明：

-   服务容器 Container 负责启动，加载，运行服务提供者。
-   服务提供者Provider在启动时，向注册中心注册自己提供的服务。
-   服务消费者Consumer在启动时，向注册中心订阅自己所需的服务。
-   注册中心Registry返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
-   服务消费者Consumer，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
-   服务消费者 Consumer和提供者 Provider，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心Monitor。

总结：

-   服务提供者注册服务到注册中心
-   服务消费者从注册中心获取服务，**并通过负载均衡策略选择服务提供者**
-   动态增减服务提供者和服务消费者

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648443640-image.png)

服务提供者、服务注册中心和服务消费者的关系

参考：

-   [在 Java 项目中配置和使用 dubbo](https://www.bilibili.com/video/BV1kp4y1x7Ad?p=16&spm_id_from=pageDriver)；
-   [dubbo 官网](http://dubbo.apache.org/zh-cn)；
-   [dubbo GitHub](https://github.com/apache/incubator-dubbo)；
-   [dubboAdmin](https://github.com/apache/incubator-dubbo-ops)：基于 web 的 dubbo 管理平台。理控制台为内部裁剪版本，开源部分主要包含：路由规则，动态配置，服务降级，访问控制，权重调整，负载均衡等管理功能。

## 异步化消息服务

同步通信和异步通信通常用来形容一次方法调用。

同步通信方法：调用一旦开始，调用者必须等到方法调用返回后，才能继续后续的行为。

异步通信方法：调用更像是消息传递，一旦开始，方法调用就会立即返回，调用者就可以继续后续的操作异步方法通常会在另一个线程中“真实”地执行，整个过程不会阻碍调用者的工作。

异步化好处：

-   不会阻塞原来的业务
-   服务调用之间解偶，无需互相关注感知

示例：

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648465141-image.png)

异步消息示例

消息服务分类：

-   JMS (Apache ActiveMQ)
-   Kafka（流式处理）
-   RocketMQ（分布式一致性）

### 消息队列通信的模式

#### 点对点模式

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648467159-image.png)

消息队列通信的模式：点对点模式

点对点模式通常是基于拉取或者轮询的消息传送模型，这个模型的特点是发送到队列的消息被一个且只有一个消费者进行处理。生产者将消息放入消息队列后，由消费者主动的去拉取消息进行消费。点对点模型的的优点是消费者拉取消息的频率可以由自己控制。但是消息队列是否有消息需要消费，在消费者端无法感知，所以在消费者端需要额外的线程去监控。

#### 发布订阅模式

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648467226-image.png)

消息队列通信的模式：发布订阅模式

发布订阅模式是一个基于消息的消息传送模型，该模型可以有多种不同的订阅者。生产者将消息放入消息队列后，队列会将消息推送给订阅过该类消息的消费者。由于是消费者被动接收推送，所以无需感知消息队列是否有待消费的消息！但是consumer1、consumer2、consumer3由于机器性能不一样，所以处理消息的能力也会不一样，但消息队列却无法感知消费者消费的速度！所以推送的速度成了发布订阅模模式的一个问题！假设三个消费者处理速度分别是8M/s、5M/s、2M/s，如果队列推送的速度为5M/s，则consumer3无法承受！如果队列推送的速度为2M/s，则consumer1、consumer2会出现资源的极大浪费！

### JMS

ActiveMQ 是Apache出品，最流行的、功能强大的即时通讯和集成模式的开源服务器。ActiveMQ 是一个完全支持JMS1.1和J2EE 1.4规范的 JMS Provider实现。提供客户端支持跨语言和协议，带有易于在充分支持JMS 1.1和1.4使用J2EE企业集成模式和许多先进的功能。

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648470208-image.png)

ActiveMQ 发布订阅模式

点对点：发送者，接收者，一对一发送，每个消息都被发送到一个特定的队列，接收者从队列中获取消息。队列保留着消息，直到他们被消费或超时。

发布订阅：客户端将消息发送到主题，消息队列存放主题，订阅者消费主题消息。

ActiveMQ存在的问题：

-   消息在磁盘中存储可能是不连续的，在磁盘存储方面损耗较大，可扩展性很差；
-   向 Consumer 推消息时忽略了 Consumer 的性能、处理速度等因素，Consumer无法控制消费消息的节奏。

参考：[ActiveMQ 即时通讯服务浅析](https://www.cnblogs.com/hoojo/p/active_mq_jms_apache_activeMQ.html)

### Kafka

Kafka是最初由Linkedin公司开发，是一个分布式、支持分区的（partition）、多副本的（replica），基于zookeeper协调的分布式消息系统，它的最大的特性就是可以实时的处理大量数据以满足各种需求场景：比如基于hadoop的批处理系统、低延迟的实时系统、storm/Spark流式处理引擎，web/nginx日志、访问日志，消息服务等等，用scala语言编写，Linkedin于2010年贡献给了Apache基金会并成为顶级开源项目。

发布订阅：客户端将消息发送到主题，消息队列存放主题，订阅者消费主题消息，**消息持久化到队尾，消费通过客户端指针**，吞吐量高。

Kafka的特性：

-   高吞吐量、低延迟：kafka每秒可以处理几十万条消息，它的延迟最低只有几毫秒。
-   可扩展性：kafka集群支持热扩展。
-   持久性、可靠性：消息被持久化到本地磁盘，并且支持数据备份防止数据丢失。
-   容错性：允许集群中节点失败（若副本数量为n,则允许n-1个节点失败）。
-   高并发：支持数千个客户端同时读写。

Kafka场景应用：

-   日志收集：可以用Kafka可以收集各种服务的log，通过kafka以统一接口服务的方式开放给各种consumer，例如hadoop、Hbase、Solr等。
-   消息系统：解耦和生产者和消费者、缓存消息等。
-   用户活动跟踪：Kafka经常被用来记录web用户或者app用户的各种活动，如浏览网页、搜索、点击等活动，这些活动信息被各个服务器发布到kafka的topic中，然后订阅者通过订阅这些topic来做实时的监控分析，或者装载到hadoop、数据仓库中做离线分析和挖掘。
-   运营指标：Kafka也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集中反馈，比如报警和报告。
-   流式处理：比如spark streaming和storm。
-   事件源

Kafka一些重要设计思想：

-   Consumergroup：各个consumer可以组成一个组，每个消息只能被组中的一个consumer消费，如果一个消息可以被多个consumer消费的话，那么这些consumer必须在不同的组。
-   消息状态：在Kafka中，消息的状态被保存在consumer中，broker不会关心哪个消息被消费了被谁消费了，只记录一个offset值（指向partition中下一个要被消费的消息位置），这就意味着如果consumer处理不好的话，broker上的一个消息可能会被消费多次。
-   消息持久化：Kafka中会把消息持久化到本地文件系统中，并且保持极高的效率。
-   消息有效期：Kafka会长久保留其中的消息，以便consumer可以多次消费，当然其中很多细节是可配置的。
-   批量发送：Kafka支持以消息集合为单位进行批量发送，以提高push效率。
-   push-and-pull :Kafka中的Producer和consumer采用的是push-and-pull模式，即Producer只管向broker push消息，consumer只管从broker pull消息，两者对消息的生产和消费是异步的。
-   Kafka集群中broker之间的关系：不是主从关系，各个broker在集群中地位一样，我们可以随意的增加或删除任何一个broker节点。
-   负载均衡方面： Kafka提供了一个 metadata API来管理broker之间的负载（对Kafka0.8.x而言，对于0.7.x主要靠zookeeper来实现负载均衡）。
-   同步异步：Producer采用异步push方式，极大提高Kafka系统的吞吐率（可以通过参数控制是采用同步还是异步方式）。
-   分区机制partition：Kafka的broker端支持消息分区，Producer可以决定把消息发到哪个分区，在一个分区中消息的顺序就是Producer发送消息的顺序，一个主题中可以有多个分区，具体分区的数量是可配置的。分区的意义很重大，后面的内容会逐渐体现。
-   离线数据装载：Kafka由于对可拓展的数据持久化的支持，它也非常适合向Hadoop或者数据仓库中进行数据装载。
-   插件支持：现在不少活跃的社区已经开发出不少插件来拓展Kafka的功能，如用来配合Storm、Hadoop、flume相关的插件。

#### Kafka基本原理

##### 基础架构

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648467459-image.png)

Kafka 基础架构

-   Producer：Producer即生产者，消息的产生者，是消息的入口。
-   Broker：Broker是kafka实例，每个服务器上有一个或多个kafka的实例，我们姑且认为每个broker对应一台服务器。每个kafka集群内的broker都有一个不重复的编号，如图中的broker-0、broker-1等……
-   Topic：消息的主题，可以理解为消息的分类，kafka的数据就保存在topic。在每个broker上都可以创建多个topic。
-   Partition：Topic的分区，每个topic可以有多个分区，**分区的作用是做负载，提高kafka的吞吐量**。同一个topic在不同的分区的数据是不重复的，**partition的表现形式就是一个一个的文件夹**！
-   Replication:每一个分区都有多个副本，副本的作用是做备份。当主分区（Leader）故障的时候会选择一个备份（Follower），成为Leader。在kafka中默认副本的最大数量是10个，且副本的数量不能大于Broker的数量，**follower和leader绝对是在不同的机器，同一机器对同一个分区也只可能存放一个副本**（包括自己）。
-   Message：每一条发送的消息主体。
-   Consumer：消费者，即消息的消费方，是消息的出口。
-   Consumer Group：我们可以将多个消费者组成一个消费组，在kafka的设计中**同一个分区的数据只能被消费组中的某一个消费者消费**。同一个消费组的消费者可以消费同一个topic的不同分区的数据，这也是为了提高kafka的吞吐量！
-   Zookeeper：kafka集群**依赖zookeeper来保存集群的的元信息**，来保证系统的可用性。

##### 工作流程分析

（一）发送数据：

producer就是生产者，是数据的入口。**Producer在写入数据的时候永远的找leader**，不会直接将数据写入**follower**！  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200624150617430.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM2NjQ5OQ==,size_16,color_FFFFFF,t_70)  
消息写入leader后，**follower是主动的去leader进行同步**的！producer采用push模式将数据发布到broker，每条消息追加到分区中，顺序写入磁盘，所以保证同一分区内的数据是有序的！写入示意图如下：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200624150636117.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM2NjQ5OQ==,size_16,color_FFFFFF,t_70)  
上面说到数据会写入到不同的分区，那kafka为什么要做分区呢？分区的主要目的是：

1.  **方便扩展**：因为一个topic可以有多个partition，所以我们可以通过扩展机器去轻松的应对日益增长的数据量。
2.  **提高并发**：以partition为读写单位，可以多个消费者同时消费数据，提高了消息的处理效率。

如果某个topic有多个partition，producer又怎么知道该将数据发往哪个partition呢？kafka中有几个原则：

1.  partition在写入的时候可以指定需要写入的partition，如果有指定，则写入对应的partition。
2.  如果没有指定partition，但是设置了数据的key，则会根据key的值hash出一个partition。
3.  如果既没指定partition，又没有设置key，则会轮询选出一个partition。

保证消息不丢失是一个消息队列中间件的基本保证，那producer在向kafka写入消息的时候，怎么保证消息不丢失呢？通过**ACK应答机制**！在生产者向队列写入数据的时候可以设置参数来确定是否确认kafka接收到数据，这个参数可设置的值为**0、1、all**。

-   0代表producer往集群发送数据不需要等到集群的返回，不确保消息发送成功。安全性最低但是效率最高。
-   1代表producer往集群发送数据只要leader应答就可以发送下一条，只确保leader发送成功。
-   all代表producer往集群发送数据需要所有的follower都完成从leader的同步才会发送下一条，确保leader发送成功和所有的副本都完成备份。安全性最高，但是效率最低。

最后要注意的是，如果往不存在的topic写数据，能不能写入成功呢？kafka会自动创建topic，分区和副本的数量根据默认配置都是1。

（二）保存数据：

Producer将数据写入kafka后，集群就需要对数据进行保存了！kafka将数据保存在磁盘，可能在我们的一般的认知里，写入磁盘是比较耗时的操作，不适合这种高并发的组件。Kafka初始会单独开辟一块磁盘空间，顺序写入数据（效率比随机写入高）。

（1）Partition 结构

前面说过了每个topic都可以分为一个或多个partition，如果你觉得topic比较抽象，那partition就是比较具体的东西了！Partition在服务器上的表现形式就是一个一个的文件夹，**每个partition的文件夹下面会有多组segment文件**，**每组segment文件又包含.index文件、.log文件、.timeindex文件**（早期版本中没有）三个文件， **log文件就实际是存储message的地方，而index和timeindex文件为索引文件，用于检索消息**。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200624170905606.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM2NjQ5OQ==,size_16,color_FFFFFF,t_70)  
如上图，这个partition有三组segment文件，**每个log文件的大小是一样的，但是存储的message数量是不一定相等的**（每条的message大小不一致）。**文件的命名是以该segment最小offset来命名的，如000.index存储offset为0~368795的消息，kafka就是利用分段+索引的方式来解决查找效率的问题。**

（2）Message结构

上面说到log文件就实际是存储message的地方，我们在producer往kafka写入的也是一条一条的message，那存储在log中的message是什么样子的呢？**消息主要包含消息体、消息大小、offset、压缩类型**……等等！我们重点需要知道的是下面三个：

-   **offset**：offset是一个占8byte的有序id号，它可以唯一**确定每条消息在parition内的位置**！
-   **消息大小**：消息大小占用4byte，用于描述消息的大小。
-   **消息体**：消息体存放的是实际的消息数据（被压缩过），占用的空间根据具体的消息而不一样。

（3）存储策略

无论消息是否被消费，kafka都会保存所有的消息。那对于旧数据有什么删除策略呢？

-   基于时间，默认配置是168小时（7天）。
-   基于大小，默认配置是1073741824。

需要注意的是，**kafka读取特定消息的时间复杂度是O(1)**，所以这里**删除过期的文件并不会提高kafka的性能**！

（三）消费数据

消息存储在log文件后，消费者就可以进行消费了。Kafka采用的是发布订阅模式，消费者去kafka集群拉取消息，与producer相同的是，消费者在拉取消息的时候也是找**leader**去拉取。

多个消费者可以组成一个消费者组（consumer group），每个消费者组都有一个组id！同一个消费组的消费者可以消费同一topic下不同分区的数据，但是不会组内多个消费者消费同一分区的数据！  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200624171149382.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM2NjQ5OQ==,size_16,color_FFFFFF,t_70)  
如果消费者组内的消费者小于partition数量，会出现某个消费者消费多个partition数据的情况，消费的速度也就不及只处理一个partition的消费者的处理速度！如果是消费者组的消费者多于partition的数量，那会不会出现多个消费者消费同一个partition的数据呢？上面已经提到过不会出现这种情况！多出来的消费者不消费任何partition的数据。所以在实际的应用中，**建议消费者组的consumer的数量与partition的数量一致**！

查找消息的时候是怎么利用segment+offset配合查找的呢？假如现在需要查找一个offset为368801的message是什么样的过程呢？我们先看看下面的图：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200624171235860.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM2NjQ5OQ==,size_16,color_FFFFFF,t_70)

1.  先找到offset的368801message所在的segment文件（利用二分法查找），这里找到的就是在第二个segment文件。
2.  打开找到的segment中的.index文件（也就是368796.index文件，该文件起始偏移量为368796+1，我们要查找的offset为368801的message在该index内的偏移量为368796+5=368801，所以这里要查找的相对offset为5）。由于该文件采用的是**稀疏索引**的方式存储着相对offset及对应message物理偏移量的关系，所以直接找相对offset为5的索引找不到，这里同样利用二分法查找相对offset小于或者等于指定的相对offset的索引条目中最大的那个相对offset，所以找到的是相对offset为4的这个索引。
3.  根据找到的相对offset为4的索引确定message存储的物理偏移位置为256。打开数据文件，从位置为256的那个地方开始顺序扫描直到找到offset为368801的那条Message。

这套机制是建立在offset为有序的基础上，利用segment+有序offset+稀疏索引+二分查找+顺序查找等多种手段来高效的查找数据！至此，消费者就能拿到需要处理的数据进行处理了。那每个消费者又是怎么记录自己消费的位置呢？在早期的版本中，消费者将消费到的offset维护zookeeper中，consumer每间隔一段时间上报一次，这里容易导致重复消费，且性能不好！在新的版本中消费者消费到的offset已经直接维护在kafk集群的\_\_consumer\_offsets这个topic中！

### RocketMQ

发布订阅：客户端将消息发送到主题，消息队列存放主题，订阅者消费主题消息，消息队列维护高可用，并支持事务回溯机制。

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648471432-image.png)

Kafka/RocketMQ 发布订阅模式

参考：

-   [大写的服，看完这篇你还不懂RocketMQ算我输](https://www.cnblogs.com/yinjihuan/p/13672474.html) 

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
