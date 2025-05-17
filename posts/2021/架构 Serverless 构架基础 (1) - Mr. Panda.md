---
date: 2022-01-01
title: Serverless 构架基础 (1) - Mr. Panda
# category: 
# tags: 
# description:
---

# [架构] Serverless 构架基础 (1) - Mr. Panda

---
2014 年国外 Serverless 生态迅速发展，诞生了如 Serverless Framework、Vercel 等很多优秀的产品；2017 年阿里云和腾讯云发布了国内的 Serverless 产品：函数计算和云函数。Serverless 的开发框架、WEBIDE也陆续出现。

## 前言

### 课程设计

-   概念篇
-   开发基础篇
-   开发进阶篇
-   场景案例篇

### Roadmap

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639324652-image.png)

基础概念

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639324711-image.png)

应用开发

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639324843-image.png)

场景案例

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639324876-image.png)

开发进阶

## 前因后果：Serverless 架构兴起的必然因素是什么？

### Serverless 架构兴起

主流云服务商推出 Serverless 相关的云产品和新功能：

-   AWS Lambda、阿里云函数计算、腾讯云云函数各种关于 Serverless 的商业和开源产品也层出不穷
-   Serverless Framework、Openfaas、kubeless

Serverless 为什么这么火？

云计算的发展史就是 Serverless 的兴起史：**物理机时代**、**虚拟机时代**、**容器时代**、**Serverless 时代**。

### 物理机时代

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639325344-image.png)

物理机时代1

-   通过时间片轮转的方式把一个操作系统给多个用户使用。
-   云计算：一种新的计算范式，其中计算的边界将由经济原理决定，而不仅仅是技术限制。云计算不只是虚拟化技术，还是云服务商提供计算资源，使用者购买计算资源。

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639325525-image.png)

物理机时代网站部署架构

物理机时代，网站上线和稳定运行面临的最大问题就是服务器等硬件问题。

### **虚拟机时代**

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639325713-image.png)

物理机时代2

-   2001 年 Vmware 带来的针对 x86 服务器的虚拟化产品使虚拟机逐渐普及；通过虚拟化技术，它可以把一台物理机分割成多台虚拟机提供给多用户使用充分利用硬件资源，而且速度和弹性也远超物理机
-   因此也出现了很多基于虚拟化的云厂商和产品。AWS 的 EC2、阿里云 ECS、Azure Virtual Machines，这种云计算形态也被叫作 laaS（软件即服务）。

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639325933-image.png)

虚拟机时代网站部署架构

-   服务器就只负责处理用户的请求。
-   **把计算和存储分离开来**，既降低了系统负载，也提升了数据安全性。
-   **单机应用升级为了集群应用**，通过**负载均衡**，会把用户流量均匀分配到每台服务器上。

### 容器技术

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639326156-image.png)

容器时代

-   Docker 的发布代表着容器技术替代了虚拟化技术，云计算进入容器时代；容器就是把代码和运行环境打包在一起，这样代码就可以在任何地方运行
-   当容器多了的时候，如何管理就成了一个问题，于是出现了容器编排技术，比如 2014 年 Google 开源的 Kubernetes。
-   部署网站的方式发生改变：搭建 Kubernetes 集群、构建容器镜像、部署镜像。

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639326504-image.png)

容器时代网站部署架构

出现的问题：

-   需要去规划节点和 Pod 的 CPU、内存、磁盘等资源。
-   需要编写复杂的 YAML 去部署 Pod、服务，需要经常排查 Pod 出现的异常。
-   需要学习专业的运维知识。

### Serverless 时代

Serverless 指构建和运行不需要服务器管理的一种概念。

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639326756-image.png)

Serverless 的实现

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639326799-image.png)

Serverless时代网站部署架构

### 小结

物理机时代：

-   2000 年之前，我们需要通过物理机部署网站

虚拟机时代：

-   2000 年之后，虚拟化技术发展成熟，云计算行业蓬勃发展
-   我们可以基于 IaaS 和 PaaS 部署应用，提高稳定性

容器时代：

-   2013 年云计算进入容器时代，我们可以通过容器技术打包应用及运行依赖，不用关心运行环境

Serverless 时代：

-   近几年，云计算进入 Serverless 时代，我们不再需要关心服务器
-   应用也天然具有弹性

## 概念新知：到底什么是 Serverless?

### 广义的 Serverless

广义的 Serverless 是指构建和运行软件时不需要关心服务器的一种架构思想，基于 Serverless 思想实现的软件架构就是 Serverless 架构。Serverful 架构具有备份容灾、弹性伸缩、日志监控等问题，而 Serverless 就是为了解决这些问题诞生的。

### Serverless 和 Serverful 的架构区别

-   资源分配：不用关心应用运行的资源，只提供一份代码就行。
-   计费方式：按实际使用量计费，计费粒度也精确到了毫秒级。
-   弹性伸缩：可以快速根据业务并发扩容更多的实例，甚至允许缩容到零实例状态来实现零费用。

一个应用如果是 Serverless 架构的，必须要实现自动弹性伸缩和按量付费。

### 狭义的 Serverless

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639327799-image.png)

FaaS+BaaS

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639327979-image.png)

FaaS

-   Faas的特点：不用运维、事件驱动、按量付费、弹性伸缩。
-   BaaS(Backend as a servece)：BaaS 本质上就是把后端功能封装起来以接口的形式提供服务。常见的 BaaS 产品有 AWS Dynamodb、阿里云表格存储、消息中间件等这些服务都可以通过 API 进行访问。
-   基于 FaaS 和 BaS 的架构是一种计算和存储分离的架构。

### 什么不是 Serverless

**PaaS（平台即服务）**云计算虚拟机时代的主要形态之一。

-   付费标准：**按资源付费**，而不是按实际使用量付费。
-   弹性伸缩：只能针对底层的服务器进行扩缩容。

Kubernetes 本身也不是 Serverless，只是在概念方面有些类似。

-   Kubernetes 是一种容器编排技术，基于 Kubernetes，你能很方便地进行 Pod 的管理，并且**实现应用的弹性伸缩**。
-   从运维的角度来看，主流的 Kubernetes 服务提供商，提供的都是 Kubernetes 集群托管和运维服务。
-   从成本的角度来看，Kubernetes 按照**资源数量**计费。
-   Kubernetes 是介于 Serverful 和 Serverless 中间的产物。

**云原生**指的是原生为云设计的架构模式。

-   Serverless 是云原生的一种实现，云原生的另一种实现是 Kubernetes。

### Serverless 的优缺点

Serverless 的优点:

-   不用运维
-   弹性伸缩
-   节省成本
-   开发简单
-   降低风险
-   易于扩展

Serverless 的缺点：

-   依赖第三方服务
-   底层硬件资源多样性
-   应用性能瓶颈
-   函数通信效率低：FaaS 还没有好的数据通信方案
-   开发调试复杂

### 总结

-   广义上来讲：Serverless 是一种架构思想。
-   狭义上来讲：Serverless 是 FaaS 和 BaaS 的组合。
-   Serverless 架构的主要特点是**按量付费、弹性伸缩、不用运维**。

## 基础入门：开发 Serverless 应用

### 选择适合的 FaaS 平台

FaaS 产品对比图：

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639794119-image.png)

FaaS 产品对比图

-   FaaS 平台都支持 Nodejs、Python、Java 等编程语言。
-   FaaS 平台都支持 HTTP 和定时触发器。
-   FaaS 的计费都差不多，且每个月都提供一定的免费额度。

### 怎么用 FaaS 去开发 Serverless 应用呢？

#### 开发流程

传统的开发流程：

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639794323-image.png)

传统的开发流程

Serverless 应用开发流程：

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639794422-image.png)

Serverless 应用开发流程

函数计算代码示例：

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639794562-image.png)

函数计算代码示例

如何解决返回结果在浏览器中以附件的方式下载？使用自定义域名绑定到你的函数上。

Lambda 代码示例：

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639794816-image.png)

Lambda 代码示例

开发一个 Serverless 应用可以分为三个步骤：代码开发 → 函数部署 → 触发器创建。

#### 开发 Serverless 应用的基础知识

入口函数：

-   Serverless 应用是由一个个函数组成的，与 main 函数对应的就是 FaaS 中的入口函数，一般名为 handler。
-   FaaS 函数可以包含多个源文件，然后按照编程语言的模块机制相互引入。
-   把业务逻辑拆分到入口函数之外。

函数定义：

-   函数定义本质上是由云厂商、触发器和编程语言等多个条件决定的，标准的函数定义是 function (event, context), event 是事件对象，触发器不同，event 的值可能不同，context 是函数上下文。
-   Nodejs 中，仅 Lambda 的入口函数支持支持异步 async 写法，其他 FaaS 平台需要入口函数有第三个参数 callback。

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639795510-image.png)

不同编程语言入口函数示例

触发器及事件对象：

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639795829-image.png)

HTTP 触发器

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639795921-image.png)

API 网关触发器

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639795995-image.png)

定时触发器

-   触发器决定了个 Serverless 应用如何提供服务，也决定了代码应该如何编写。
-   保持入口函数的简洁，这样业务代码就与触发器无关了。
-   日志输出：在 Serverless 中，日志输出和传统应用的日志输出没有太大区别，只是日志的存储和查询方式变了。
-   异常处理：异常日志。Serverless 应用中一次异常只会影响本次函数的执行，而传统应用则会影响到整个应用的运行。

### 总结

-   Serverless 的应用基本组成单位是函数，函数之间互相独立，因此 Serverless 能提高应用稳定性。
-   函数定义与触发器和编程语言相关，不同 FaaS 平台的实现不尽相同。
-   为了使代码扩展性更强，建议你将业务逻辑拆分到入口函数之外。
-   为了使应用稳定性更好，建议你编写函数代码时充分考虑程序异常。

## 运行原理：Serverless 应用如何运行

Serverless 的运行原理，本质上就是函数的运行原理。下面从调用链路、调用方式、生命周期来讲解 Serverless 的运行原理。

### 函数调用链路：事件驱动函数执行

事件驱动：浏览器、nodejs、Serverless 等。

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639796756-image.png)

Serverless 的事件驱动机制

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639796850-image.png)

Serverless 函数调用链路图

### 函数调用方式：同步调用和异步调用

同步调用：

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639796942-image.png)

同步调用

异步调用：

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639796986-image.png)

异步调用

-   对于函数计算来说，定时触发器就是异步调用的。
-   异步调用怎么重试？对于异步调用，FaaS 会进行有限次重试，如果你关心调用结果的正确性，可以为函数配置“**异步调用目标**”，将调用结果发送到消息队列或其他服务中，通过监听消息得到异步执行结果。

### 函数生命周期：冷启动与热启动

函数启动过程：

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639797413-image.png)

函数启动过程

函数请求示意图：

![](https://www.jonsam.site/wp-content/uploads/2021/12/1639797528-image.png)

函数请求示意图

-   **重复执行上下文**：串联请求中，后面的请求会使函数热启动。如果函数每分钟都执行，则函数几乎都是热启动的，也就是会重复使用之前的执行上下文。

### 总结

-   传统应用部署：直接进行处理，无需启动应用、一直消耗硬件资源；
-   Serverless 的优势：应用的百毫秒启动、资源利用率和业务性能的平衡。
-   组成 Serverless 应用的函数是事件驱动的，但也可以直接同 AP 调用函数。
-   函数可以同步调用或异步调用，定时触发器函数是异步调用的，异步调用函数建议主动记录并处理异步调用结果。
-   函数的启动过程分为下载代码、启动容器、启动运行环境、执行代码四个步骤，前三个步骤称为冷启动，最后一个步骤是热启动。
-   执行上下文重用可以提高 Serverless 应用性能，但在编写代码时要注意执行上下文重用带来的风险。

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
