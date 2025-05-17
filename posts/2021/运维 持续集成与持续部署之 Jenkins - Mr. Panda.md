---
date: 2021-01-01
title: 持续集成与持续部署之 Jenkins - Mr. Panda
# category: 
# tags: 
# description:
---

# [运维] 持续集成与持续部署之 Jenkins - Mr. Panda

---
持续集成是一种软件开发实践_。在持续集成中，团队成员频繁集成他们的工作成果。每次集成会经过自动构建（包括自动测试）的检验，以尽快发现集成错误。_本文的主要内容是 CI/CD 的重要工具 Jenkins 的学习和使用，内容包括使用简介、安装方式、环境配置、前端应用等。

## 使用简介

Jenkins 是开源 CI&CD 软件领导者，提供超过 1000 个插件来支持构建、部署、自动化，满足任何项目的需要。

Jenkins 的特点：

-   持续集成和持续交付：作为一个可扩展的自动化服务器，Jenkins 可以用作简单的 CI 服务器，或者变成任何项目的**持续交付中心**。
-   简易安装：Jenkins 是一个**基于 Java 的独立程序**，可以立即运行，包含 Windows、Mac OS X 和其他类 Unix 操作系统。
-   配置简单：Jenkins 可以通过其**网页界面轻松设置和配置**，其中包括即时错误检查和内置帮助。
-   插件：通过更新中心中的 1000 多个插件，Jenkins **集成了持续集成和持续交付工具链**中几乎所有的工具。
-   扩展：Jenkins 可以通过其**插件架构进行扩展**，从而为 Jenkins 可以做的事提供几乎无限的可能性。
-   分布式：Jenkins 可以轻松地在多台机器上分配工作，帮助更快速地**跨多个平台推动构建、测试和部署**。

相关概念：

-   流水线：_Jenkins Pipeline_（或简称为“**Pipeline**“）是一套插件，**将持续交付的实现和实施集成到 _Jenkins_** 中。_Jenkins Pipeline_ 提供了一套可扩展的工具，用于将“简单到复杂“的交付流程实现为“**持续交付即代码**”。_Jenkins Pipeline_ 的定义通常被写入到一个文本文件（称为 _Jenkinsfile_）中，该文件可以被放入项目的源代码控制库中。
-   节点：节点是一个机器，主要用于执行 _jenkins_ 任务。
-   阶段：定义不同的执行任务，比如：构建、测试、发布（部署）。
-   步骤：相当于告诉 _Jenkins_ 现在要做些什么，比如 _shell_ 命令。

## 安装方式

### 环境要求

-   机器要求：256MB 内存，建议大于 512MB；10 GB 的硬盘空间（用于 Jenkins 和 Docker 镜像）。
-   需要安装以下软件：Java8 (JRE 或者 JDK 都可以）；Docker。

### 常规安装

-   安装 Java JDK：下载对应的操作系统的 JDK，然后解压进行安装。

![](https://www.jonsam.site/wp-content/uploads/2021/12/1640929782-image.png)

linux 安装 Java JDK

-   安装 Jenkins：下载 Jenkins 最新的 war 包并在 java 环境中执行。

![](https://www.jonsam.site/wp-content/uploads/2021/12/1640929908-image.png)

安装 Jenkins

打开相应的端口并使用终端打印的默认管理员密码登录进行初始化配置即可。

### 使用 Docker 安装

-   安装 _docker_

![](https://www.jonsam.site/wp-content/uploads/2021/12/1640930712-image.png)

安装 docker

-   配置 Docker 镜像加速，使用阿里云容器加速服务。加速器会显示为你独立分配的加速地址，例如：_\[系统分配前缀\].mirror.aliyuncs.com_。

使用配置文件_`/ete/docker/daemon.json`_（没有时新建该文件）:

```json
{registry-mirrors: ['adress']}
```

```bash
systemctl daemon-reload 
systemctl restart docker 
```

安装 `Docker-compose.ym`l 文件（可选）:

![](https://www.jonsam.site/wp-content/uploads/2021/12/1640931425-image.png)

安装 Docker-compose

-   安装 Jenkins。版本可选择 _Jenkeins_ 和 _Jenkins Blue Ocean_。

> Blue Ocean 重新思考Jenkins的用户体验，从头开始设计Jenkins Pipeline, 但仍然与自由式作业兼容，Blue Ocean减少了混乱而且进一步明确了团队中每个成员。Blue Ocean 的主要特性包括：持续交付(CD)Pipeline的 **复杂可视化** ，可以让您快速直观地理解管道状态。
> 
> **Pipeline 编辑器** - 引导用户通过**直观的、可视化**的过程来创建Pipeline，从而使Pipeline的创建变得平易近人。
> 
> **个性化** 以适应团队中每个成员不同角色的需求。
> 
> 在需要干预和/或出现问题时 **精确定位** 。 Blue Ocean 展示 Pipeline中需要关注的地方， 简化异常处理，提高生产力。
> 
> **本地集成分支和合并请求**, 在与GitHub 和 Bitbucket中的其他人协作编码时实现最大程度的开发人员生产力。

![](https://www.jonsam.site/wp-content/uploads/2021/12/1640932554-image.png)

通过 Docker 安装 Jenkins

可通过 `docker logs jenkins` 查看 jenkins 打印的日志信息以获取默认的登录密码。

### 配置 Jenkins 插件加速

进入 jenkins 系统管理 → 插件管理 → 高级选项卡 → 升级站点，使用清华源。

```bash
https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
```

### 环境配置

-   Jenkins 的 URL 路径
-   全局工具的配置：Docker,、JDK(JAVA)。

### 用户权限配置

-   矩阵权限的配置
-   给管理员用户添加所有的权限
-   添加 _Authorize Project_ 插件，并且在系统管理中进行配置。配置逻辑就是给用户当前项目的矩阵权限。

### 配置自动化任务

两种执行方法：

1.  配置自由风格的项目
2.  配置 Pipeline 使用 jenkinsfile

需要注意的地方:

-   SSH 插件: SSH、SSH Agent、SSH Pipeline Steps、Publish Over SSH。
-   git 相关插件: Gitlab、Github。

管理员界面配置：

Settings -> network -> outbound requests: Allow requests to the local network from hooks and services 进行勾选.

### 前端项目中的应用

插件：

-   nodeJs 插件：主要是用于不同版本的 Node 打包。
-   Publish Over SSH：用于构建完成之后，推送到远程的 web 服务器。

Jenkins 发布到 Nginx Docker 容器：

![](https://www.jonsam.site/wp-content/uploads/2021/12/1640938030-image.png)

Pipeline 演示：

![](https://www.jonsam.site/wp-content/uploads/2021/12/1640938068-image.png)

思路：

1.  使用一台单独的 Nginx 服务器发布，使用 Publish Over SSH 插件上传。
2.  使用 Docker 在本地发布或者远程发布。
3.  使用 Dockerfile 进行镜像内的构建，使用 docker 镜像进行发布。

## TravisCI

### 使用简介

Travis CI 只支持 Github，不支持其他代码托管服务。这意味着，你必须满足以下条件，才能使用 Travis CI：拥有 Github 帐号、该帐号下面有一个项目、该项目里面有可运行的代码、该项目还包含构建测试脚本。

Travis 的步骤：github 授权及面板、获取 github 的 Tokens、配置项目 _. travis.yml_（Node 项目、Script 脚本、部署到 github pages、钩子用法）。

### 配置 NodeJS 应用

配置一个 Vue 实例并发布到 github pages：

![](https://www.jonsam.site/wp-content/uploads/2021/12/1640946767-image.png)

.travis.yml

## CircleCI

### 配置 NodeJS 应用

.`circleci/config.yml` 配置文件:

![](https://www.jonsam.site/wp-content/uploads/2021/12/1640948549-image.png)

.circleci/config.yml

## 扩展知识

### 自动化流程的发展趋势

-   集中化

以集群为基础，服务采用 **SaaS** 方式进行交付。所有的构建、测试、发布全集中进行管理。

-   微服务+无服务的应用模式

无服务器风格的架构（Serverless architecture）把 Devops 技术在服务领域的应用推向极致。当应用程序执行环境的管理被新的编程模型和平台取代后，团队的交付生产效率得到了进一步的提升。一方面它免去了很多环境管理的工作，包括设备、网络、主机以及对应的软件和配置工作，使得软件运行时环境更加稳定。另一方面，它大大降低了团队采用 Devops 的技术门槛。

在微服务端到端交付流程上，Netflix 开源了自家的 _Spinnaker_, Netflix 作为服务实践的先锋，不断推出新的开源工具来弥补社区中微服务技术和最佳实践的缺失。而 Spring Cloud 则为开发者提供了一系列工具，以便他们在所熟悉的 Spring 技术栈下使用这些服务协调技术（coordination techniques），如服务发现、负载均衡、熔断和健康检查。

-   人工智能领域的应用

Devops 的最早实践来自于互联网企业的 Web 应用，相应的思想被引入企业级应用并促进了一系列工具的发展。在人工智能领域，Tensorflow 就是这样一个例子，它可以有多种 Devops 友好的安装和部署方式，例如采用 Docker 进行部署。

随着 Python 在大数据、人工智能、区块链、服务以及 Docker 中的发展，可以预见 Python 在日后的领域仍然会发挥重要的作用。

-   安全推动 Devops 的发展

安全是 Devops 永远绕不开的话题，也往往是新技术在传统行业（例如金融和电信）应用中的最大阻碍。一方面，组织结构的转型迫使企业要打破原先的部门墙，这意味着很多原先的控制流程不再适用。另一方面，由于大量的 Devops 技术来源于开源社区，缺乏强大技术实力的企业在应用相关技术时不免会有所担忧。

### 复杂的 DevOps 相关工具

![](https://www.jonsam.site/wp-content/uploads/2021/12/1640949592-image.png)

复杂的 DevOps 相关工具

## 参考链接

-   [Jenkins 开发文档](https://www.jenkins.io/zh/doc/)；

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
