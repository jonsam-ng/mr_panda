---
date: 2022-01-01
title: Docker 技术入门基础 (1) - 容器技术概述 - Mr. Panda
# category: 
# tags: 
# description:
---

# [运维] Docker 技术入门基础 (1) - 容器技术概述 - Mr. Panda

---
本教程是运维教程的一部分，什么是Docker？

Docker 是一个开源的**应用容器引擎**，让开发者可以打包他们的应用以及依赖包到一个可移植的镜像(Docker Image)中，然后发布到任何流行的 Linux或Windows 机器上，也可以实现[**虚拟化**](https://baike.baidu.com/item/%E8%99%9A%E6%8B%9F%E5%8C%96/547949)。容器是完全使用**[沙箱](https://baike.baidu.com/item/%E6%B2%99%E7%AE%B1/393318)机制**，相互之间不会有任何接口。

Docker 技术学习目录：

1.  容器技术概述
2.  Docker容器安装部署
3.  Docker 镜像管理
4.  Docker 基础操作
5.  Docker 高级操作
6.  Docker File详解
7.  综合实验
8.  Docker 的网络模型

**什么是容器？**

容器是一种容纳、存储、运输物品的基础工具。类似于集装箱。

**虚拟化技术？**

![](https://www.jonsam.site/wp-content/uploads/2021/05/1622392833-image.png)

虚拟化分为寄居虚拟化和裸机虚拟化，此图展示的是寄居虚拟化的结构，物理主机承载的OS上安装VM工具，借此达到虚拟多个环境的目的。详见：[虚拟化技术详解](https://blog.csdn.net/gui951753/article/details/81045508)。

虚拟化即使可以实现这种场景：让程序运行在同一台宿主机上的相对隔离的环境中，互不干扰。但是这种方法花销比较大，实现起来比较繁琐。为运行一个程序安装一个独立的操作系统不太客观。而Docker技术完美地解决了这种问题，我们来看一下Docker技术的结构：

![](https://www.jonsam.site/wp-content/uploads/2021/05/1622393622-image.png)

可以看到Docker将Hypervisor成替换成了Docker Engine层，Docker Engine会自动将上层的应用运行环境进行隔离，而不需要单独为每个程序的运行提供一个完整的操作系统。

**我们所说的隔离是一种什么样的概念呢？**

下面是NameSpace资源隔离条件：

![](https://www.jonsam.site/wp-content/uploads/2021/05/1622393915-image.png)

运行程序满足NameSpace资源隔离条件，则两个程序的运行可以看做是隔离的，每个程序所运行的环境，可以看做一个独立的容器。这个可以从虚拟化的角度进行理解，容器化要解决的正是虚拟化所要解决的问题。

**使用docker容器化封装应用程序的意义**？

-   统一了基础设施环境--docker环境：硬件配置、OS版本、运行时环境异构...
-   统一了程序打包方式--docker镜像：java、python、node...
-   统一了程序部署方式--docker容器：docker run

## 容器技术简史

我们先了解一下容器化技术的简史：

chroot -> Solaris Containers -> OpenVZ -> cgroups -> LXC(Linux Containers) -> Docker(2013年) -> Rocket（2014年，不够简单、规范） -> Windows Containers -> K8S 微服务。

## Docker 诞生

参看： [维基百科：Docker](https://m.hk.guom.site/wiki/Docker)

Build Once，Run Anywhere.

## Docker 是什么

-   Docker基于容器技术的**轻量级虚拟化解决方案**。
-   Docker是容器引擎，把Linux的cgroup、namespace 等容器底层技术进行封装**为用户提供创建和管理容器的编辑界面、命令行、API**。
-   Docker是一个诞生于2013**开源**项目，基于**go语言实现**。
-   Docker引入可一整套**容器化管理生态系统**，包括分成镜像模型、容器注册库、友好的Rest Api。

## Docker结构再看

![](https://www.jonsam.site/wp-content/uploads/2021/05/1622395217-image.png)

docker结构图

-   Docker 是没有VM层的，它是直接建立在属主OS上的，通过Docker Engine 提供容器化的服务。
-   Docker为每一个应用提供独立的、隔离的运行环境。容器是隔离的，并且共享了宿主OS，并且在必要的时候还可以共享一些公共二进制文件或者是库文件。
-   Docker具有快速部署、更低载荷、简单移植、快速重启等特点。

## 容器技术和虚拟化技术的优缺点比较

![](https://www.jonsam.site/wp-content/uploads/2021/05/1622395680-image.png)

容器技术和虚拟化技术的优缺点比较

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
