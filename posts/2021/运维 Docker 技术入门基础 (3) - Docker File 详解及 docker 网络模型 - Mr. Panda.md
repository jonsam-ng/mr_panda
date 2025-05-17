---
date: 2022-01-01
title: Docker 技术入门基础 (3) - Docker File 详解及 docker 网络模型 - Mr. Panda
# category: 
# tags: 
# description:
---

# [运维] Docker 技术入门基础 (3) - Docker File 详解及 docker 网络模型 - Mr. Panda

---
这篇文章是Docker技术入门基础的一部分，包括Docker File的概念、基本结构、文件指令等，以及docker的网络模型。通过这节课学习，应该能够看懂docker file文件，并能够编写符合业务需求的docker file。学完这节课，docker的入门基础就告一段落，更多的进阶内容如数据卷操作、网络配置、安全相关、底层实现等请参考文末[docker中文文档](http://www.dockerinfo.net/document)。

## DockerFile

Docker镜像制作的途径：

-   docker commit
-   docker file

加入了docker file后，我们来看下docker整体的结构：

![](https://www.jonsam.site/wp-content/uploads/2021/06/1623508552-docker-%E7%BB%93%E6%9E%84%E6%A2%B3%E7%90%86.png)

docker整体结构图

### 什么是dockerfile?

Dockerfile是一个**包含用于组合映像的命令的文本文档**。可以使用在命令行中调用任何命令。 Docker通过读取`Dockerfile`中的指令自动生成映像。

`docker build`命令用于从Dockerfile构建映像。可以在`docker build`命令中使用`-f`标志指向文件系统中任何位置的Dockerfile。

```bash
docker build -f /path/to/a/Dockerfile
```

### Dockerfile的基本结构

Dockerfile 一般分为四部分：**基础镜像信息**、**维护者信息**、**镜像操作指令和容器启动时执行指令**，’#’ 为 Dockerfile 中的注释。

docker file 的规则如下：

-   ‘#’为文件注释。
-   一般指令大写，内容小写。尽管指令大小写不敏感。
-   docker file 是按照从上到下的顺序依次执行的。
-   第一个非注释指令必须是“From”指令，用于指定Base Image，后续指令运行于Base Image的环境。
-   Base Image可以是任何可用镜像文件，默认情况下，docker build会在docker主机（本地）查找指定的镜像文件，当其在本地不存在时，则会从Docker registry（远端）上拉取所需镜像文件。

### Dockerfile文件指令

4组核心的Dockerfile指令

-   USER/WORKDIR指令
-   ADD/EXPOSE指令
-   RUN/ENV指令
-   CMD/ENTRYPOINT指令

Docker以从上到下的顺序运行Dockerfile的指令。常用的指令如下：

-   FROM：指定基础镜像，必须为第一个命令；
-   MAINTAINER: 维护者信息；
-   RUN：**构建镜像时**执行的命令；
-   ADD：将本地文件添加到容器中，tar类型文件会自动解压(网络压缩资源不会被解压)，可以访问网络资源，类似wget；
-   COPY：功能类似ADD，但是是不会自动解压文件，也不能访问网络资源；
-   CMD：**构建容器后**调用，也就是**在容器启动时**才进行调用；
-   ENTRYPOINT：配置容器，使其可执行化。配合CMD可省去"application"，只使用参数；
-   LABEL：用于为镜像添加元数据；
-   ENV：设置环境变量；
-   EXPOSE：指定与外界交互的端口，只在docker run -P参数时生效；
-   VOLUME：用于指定持久化目录；
-   WORKDIR：工作目录，类似于cd命令；
-   USER:指定运行容器时的用户名或 UID（主进程使用的用户），后续的 RUN 也会使用指定用户。使用USER指定用户时，可以使用用户名、UID或GID，或是两者的组合。当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户；
-   ARG：用于指定传递给构建运行时的变量；
-   ONBUILD：用于设置镜像触发器；

### DockerFile 示例

```docker
Version 1.0 FROM centos MAINTAINER jonsam ENV PATH /usr/local/nginx/sbin:$PATH ADD nginx-1.8.0.tar.gz /usr/local/ ADD epel-release-latest-7.noarch.rpm /usr/local/ RUN rpm -ivh /usr/local/epel-release-latest-7.noarch.rpm RUN yum install -y wget lftp gcc gcc-c++ make openssl-devel pcre-devel pcre && yum clean all RUN useradd -s /sbin/nologin -M www WORKDIR /usr/local/nginx-1.8.0 RUN ./configure --prefix=/usr/local/nginx --user=www --group=www --with-http_ssl_module --with-pcre && make && make install RUN echo "daemon off;" >> /etc/nginx.conf EXPOSE 80 CMD ["nginx"]
```

## Docker网络模型

使用docker run创建Docker容器时，可以用--net选项指定容器的网络模式，Docker有以下4种网络模式：

-   host模式，使用--net=host指定。
-   container模式，使用--net=container:NAME\_or\_ID指定。联合模式，容器间共享网络空间。
-   none模式，使用--net=none指定。
-   bridge模式，使用--net=bridge指定，默认设置，可省略。

参考链接：[docker的4种网络模型](https://www.cnblogs.com/yyxianren/p/10727736.html)

## 参考链接

-   [使用Dockerfile构建Docker镜像](https://www.jianshu.com/p/cbce69c7a52f)；
-   [Docker中文文档](http://www.dockerinfo.net/document)；（更多进阶知识查看文档）

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
