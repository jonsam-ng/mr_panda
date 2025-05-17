---
date: 2022-01-01
title: 从零开始学软件架构：API 网关层架构设计 - Mr. Panda
# category: 
# tags: 
# description:
---

# [架构] 从零开始学软件架构：API 网关层架构设计 - Mr. Panda

---
本文是从零开始学软件架构系列文章的第三篇，主题为API 网关层架构设计。本文的主要内容包括：传统会话管理方式、分布式会话管理方案、接入层控制、服务调用及聚合等内容。学习目的是了解 API 网关层会话管理的方式及原理、学习分布式会话管理的解决方案，学习接入层控制的应用场景，了解服务调用及聚合重接入和轻接入的结构。

## 课程概要

-   分布式会话管理
-   接入层控制
-   服务调用及聚合

## 分布式会话管理

会话管理：

由于 http 请求的**无状态性**，因此引入了 session 会话管理机制，标识 BS 端之间**会话状态**。

分布式会话管理：

在Web项目开发中，会话管理是一个很重要的部分，用于存储与用户相关的数据。通常是由符合session规范的容器来负责存储管理，也就是一旦容器关闭，重启会导致会话失效。因此打造一个高可用性的系统，必须将session管理从容器中独立出来。

区别于传统的依赖 web server session 的会话管理状态，需要引入**集中的会话存储容器**，用于鉴别分布式状态下的 BS 端之间的会话标识。

会话管理的几种方式：

-   基于 server 端的 session 管理方式
-   基于 cookie 的管理方式
-   基于 token 的管理方式
-   安全问题

### 基于 server 端的 session 管理方式

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648392280-image.png)

基于 server 端的 session 管理方式

特点：

-   依赖 webserver 的 session 容器
-   cookie 跨域访问问题处理复杂
-   浏览器禁用 cookie 问题

### 基于 cookie 的管理方式

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648392485-image.png)

基于 cookie 的管理方式

特点：

-   cookie 跨域访问问题处理复杂
-   浏览器禁用 cookie 问题

### 基于 token 的管理方式

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648392670-image.png)

基于 token 的管理方式

特点：

-   有效存储 token，保证每次调用都能拿到 token。
-   需要应用代码处理将 token 加到 header 或接口传参内。
-   不依赖浏览器 cookie 禁用问题，移动 native 应用 hybrid 开发方式支持好。

### 安全问题

-   cookie 并非安全，被劫持概率高，XSS（跨站脚本攻击，Cross Site Scripting）, CSRF（跨站请求伪造，Cross-site request forgery） 安全攻击等。
-   token 凭证被劫持，伪造请求。
-   HTTPS 请求防泄漏。
-   风控、主动失效及过期机制。

参考：

-   [XSS和 CSRF攻击详解](https://www.jianshu.com/p/64a413ada155)

### 分布式会话管理

-   集中式的会话管理，凭证放到 Redis, MemCache，数据库等。
-   重写 session 处理方式。

#### 分布式会话管理的解决方案

-   容器扩展:

通过容器插件来实现，比如基于Tomcat的[tomcat-redis-session-manager](https://github.com/jcoleman/tomcat-redis-session-manager)，基于Jetty的[jetty-session-redis](https://github.com/Ovea/jetty-session-redis)等等。好处是对项目来说是透明的，无需改动代码。由于过于依赖容器，一旦容器升级或者更换意味着又得从新来过。并且代码不在项目中，对开发者来说维护也是个问题。

-   重写会话管理工具类

自己写一套会话管理的工具类，包括Session管理和Cookie管理，在需要使用会话的时候都从自己的工具类中获取，而工具类后端存储可以放到Redis中。很显然这个方案灵活性最大，但开发需要一些额外的时间。

-   使用框架的会话管理工具

[spring-session](http://docs.spring.io/spring-session/docs/current/reference/html5/) 既不依赖容器，又不需要改动代码，并且是用了spring-data-redis那一套连接池，可以说是最完美的解决方案。当然，前提是项目要使用Spring Framework才行。

#### 使用Spring Session

Spring Session为企业级Java应用的session管理带来了革新，使得以下的功能更加容易实现：

-   将session所保存的状态卸载到特定的外部session存储中，如Redis或Apache Geode中，它们能够以独立于应用服务器的方式提供高质量的集群。
-   当用户使用WebSocket发送请求的时候，能够保持HttpSession处于活跃状态。
-   在非Web请求的处理代码中，能够访问session数据，比如在JMS消息的处理代码中。
-   支持每个浏览器上使用多个session，从而能够很容易地构建更加丰富的终端用户体验。
-   控制session id如何在客户端和服务器之间进行交换，这样的话就能很容易地编写Restful API，因为它可以从HTTP 头信息中获取session id，而不必再依赖于cookie。

## 接入层控制

### 控制什么

通过 API 网关接入层控制程序入口处需要实现的逻辑，包括**身份验证，流量控制，路由服务，记录调试或统计信息**等。

实现原理：使用不同接入层框架的通用“Filter”功能实现，如 Java Servlet Filter、Spring MVC HandlerInterceptor、Zuul Filter。

Java Servlet Filter:

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648393761-image.png)

Java Servlet Filter

Spring MVC HandlerInterceptor:

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648393859-image.png)

Spring MVC HandlerInterceptor

Zuul Filter:

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648393877-image.png)

Zuul Filter

身份验证:

1.  通过会话管理获取登陆用户凭证
2.  通过用户凭证获取到用户身份信息
3.  验证对应 URL 是否可被对应身份的用户访问

流量控制：

1.  对应 URL 的流量是否可以承载，若不能，限流
2.  对应服务分级的流量是否可以承载，若不能，限流
3.  对应整个系统的总流量是否可以承载，若不能，限流

路由服务：

1.  根据对应 ur 的规则寻找到响应服务
2.  判定服务状态，做服务路由调用

调试或统计信息

1.  切面打印日志调试信息
2.  切面打印 cat 监控

## 服务调用及聚合

API 网关通过了接入层控制并路由后进入核心的服务调用环节，通过对后端服务的调用并聚合服务输出的数据后返回访问层。

### 分类

-   “重”接入 spring mvc+dubbo
-   “轻”接入 spring cloud zuul

#### 重接入

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648429599-image.png)

重接入结构图

优点：可以灵活的在 web 层处理业务逻辑，聚合服务。

缺点：服务单一性不够，且过度的 web 层业务聚合能力会导致服务不便于管理。

#### 轻接入

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648429818-image.png)

轻接入结构图

优点：服务单一，可提供配置化接入。

缺点：聚合服务处理不够灵活，需要由 service provider 提供聚合服务能力。

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
