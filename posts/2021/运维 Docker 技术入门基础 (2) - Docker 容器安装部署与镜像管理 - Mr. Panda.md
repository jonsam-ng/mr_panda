---
date: 2022-01-01
title: Docker 技术入门基础 (2) - Docker 容器安装部署与镜像管理 - Mr. Panda
# category: 
# tags: 
# description:
---

# [运维] Docker 技术入门基础 (2) - Docker 容器安装部署与镜像管理 - Mr. Panda

---
这篇文章是Docker技术入门基础的一部分，包括Docker安装方法、常用docker命令、docker镜像特性与docker管理基础等内容。本节的学习目标是理解镜像、容器和仓库的关系，使用常用的docker命令来管理docker镜像和容器，完成各种基础的docker操作。

## 关于Docker安装

-   下载方式：从官网下载或者从GitHub源码构建。[Download and install Docker](https://docs.docker.com/get-docker/)。
-   Docker版本：企业版（EE）和社区版（CE）。
-   支持OS：Linux、Windows、OSX
-   版本号：遵循YY.MM-xx格式，类似于Ubuntu等项目。17年之前遵循大版本号和小版本号格式。

## 常用docker命令

-   docker info：查看docker信息
-   docker run ~：启动docker镜像，-p映射端口，-v挂载数据卷，-e传递环境变量
-   docker version：查看docker 版本
-   docker pull ~：拉取镜像
-   docker help：查看帮助
-   docker stop ~：停止容器
-   docker restart ~：重启容器
-   docker rm：删除容器
-   docker commit ~：将容器的状态保存为镜像
-   docker images：查看所有镜像的列表
-   docker search ~：在registry中查找镜像
-   docker history ~：查看镜像的历史版本
-   docker push ~：将镜像推送到registry
-   docker ps -a：列出本地容器的全部进程
-   docker exec -ti ~ /bin/bash：进入容器并挂载终端
-   docker save ~ > name：导出镜像到本地磁盘
-   docker load：从本地磁盘导入镜像
-   docker logs -f ~：打印运行日志，-f 动态输出

更多命令参考：[Docker 命令大全](https://www.runoob.com/docker/docker-command-manual.html)。

![](https://www.jonsam.site/wp-content/uploads/2021/06/1622812772-1e8e957650557ead3f1915b03142de31d55d-removebg-preview.png)

docker 的三个核心概念：

1.  镜像（image）：一个`Linux`的文件系统。
2.  容器（container）：一个独立于宿主机的隔离进程，并且有属于容器自己的网络和命名空间。
3.  仓库（repo）：集中存储镜像的地方。

镜像、容器和仓库的关系：

![](https://www.jonsam.site/wp-content/uploads/2021/05/1623492893-%E5%AE%B9%E5%99%A8%E3%80%81%E9%95%9C%E5%83%8F%E3%80%81%E4%BB%93%E5%BA%93%E7%9A%84%E5%85%B3%E7%B3%BB.png)

镜像的结构：

registry名称/repo名称/镜像名称:tag名称

镜像特性：

![](https://www.jonsam.site/wp-content/uploads/2021/05/1623495241-docker%E9%95%9C%E5%83%8F%E7%89%B9%E6%80%A7.png)

-   docker镜像位于bootfs之上，其中第2层镜像为Base Image，每一层镜像与下层的镜像是父子关系。
-   所有的image层都是readonly的，只有位于顶层的container是可读可写的。

容器的生命周期：

-   检查本地是否存在镜像，如果不存在即从远端仓库检索
-   利用镜像启动容器
-   分配一个文件系统，并在只读的镜像层外挂载一层可读写层
-   从宿主机配置的网桥接口中桥接一个虚拟接口到容器
-   从地址池配置一个ip地址给容器
-   执行用户指定的指令
-   执行完毕后容器终止

`docker run` 来创建容器时，会运行容器的生命周期。

![](https://www.jonsam.site/wp-content/uploads/2021/05/1623500632-%E5%AE%B9%E5%99%A8%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)

容器的生命周期，created/paused/stopped/running/killed。

## 镜像的实现原理

Docker 镜像是怎么实现增量的修改和维护的？ 每个镜像都由很多层次构成，**Docker 使用 Union FS** 将这些**不同的层结合到一个镜像中**去。

通常 Union FS 有两个用途, 一方面可以实现不借助 LVM、RAID 将多个 disk 挂到同一个目录下,另一个更常用的就是将一个**只读的分支和一个可写的分支联合在一起**，Live CD 正是基于此方法可以允许在镜像不变的基础上允许用户在其上进行一些写操作。 Docker 在 AUFS 上构建的容器也是利用了类似的原理。

## 参考链接

-   [10分钟快速掌握Docker必备基础知识](https://juejin.cn/post/6844903918372143112)；
-   [Docker的三大核心组件：镜像，容器与仓库](https://juejin.cn/post/6844903938030845966)；

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
