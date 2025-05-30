---
date: 2022-01-01
title: 从零开始学软件架构：访问层架构设计 - Mr. Panda
# category: 
# tags: 
# description:
---

# [架构] 从零开始学软件架构：访问层架构设计 - Mr. Panda

---
本文是从零开始学软件架构系列文章的第二篇，主题为访问层架构设计。本文的主要内容包括：课程概要、LVS 接入系统、LVS 调度策略、Nginx功能原理、正向代理和反向代理、Nginx 进程模型、Nginx 应用等内容。学习目的是深入学习 LVS三种模式的原理和 LVS 调度策略，学习 Nginx 的功能和原理并且能够应用到 Nginx 的反向代理、动静分离、代理缓存的配置。

## 课程概要

-   LVS 接入系统
-   Nginx 原理
-   Nginx 应用

## LVS 接入系统

### 什么是 LVS

**LVS**是**Linux Virtual Server**的简写，意即**Linux虚拟服务器**，是一个虚拟的服务器集群系统，在1998年5月由章文嵩先生主导开发。

LVS 集群采用 **IP 负载均衡技术**和**基于内容请求分发技术**，将请求均衡地转移到不同的服务器上执行，且调度器**自动屏蔽掉服务器的故障**，从而将一组服务器构成一个**高性能的、高可用的虚拟服务器**，整个服务器**集群的结构对客户是透明的**，而且无需修改客户端和服务器端的程序。

LVS服务器可以让客户端将LVS服务器作为一个连接的单点，仅仅通过连接LVS服务器便可以得到后端一整个服务器集群的处理与存储能力，这样能够大大提高系统的扩展性与可用性，同时也能够提供服务的安全性，单一入侵一台服务器并不会破坏其他与该服务器隔离的服务。

LVS通过工作于内核的ipvs模块来实现功能，其主要工作于netfilter的INPUT链上。除此之外，还需要一个用户态工具，ipvsadm，用于用户负载集群定义和集群服务管理。

LVS 三种不同的模式：

-   LVS/NAT：Virtual Server via Network Address Translation
-   LVS/DR：Virtual Server via Direct Routing
-   LVS/TUN：Virtual Server via IP Tunneling

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648303323-image.png)

LVS 接入系统

#### 相关术语

-   LB (Load Balancer 负载均衡)
-   HA (High Available 高可用)
-   Failover (失败切换)
-   Cluster (集群)
-   LVS (Linux Virtual Server Linux 虚拟服务器)
-   DS (Director Server)，指的是前端负载均衡器节点
-   RS (Real Server)，后端真实的工作服务器
-   VIP (Virtual IP)，虚拟的IP地址，向外部直接面向用户请求，作为用户请求的目标的 IP 地址
-   DIP (Director IP)，主要用于和内部主机通讯的 IP 地址
-   RIP (Real Server IP)，后端服务器的 IP 地址
-   CIP (Client IP)，访问客户端的 IP 地址

#### LVS基本原理

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648305735-image.png)

LVS基本工作原理

1.  当用户向负载均衡调度器（Director Server）发起请求，调度器将请求发往至内核空间。
2.  PREROUTING链首先会接收到用户请求，判断目标IP确定是本机IP，将数据包发往INPUT链。
3.  IPVS是工作在INPUT链上的，当用户请求到达INPUT时，IPVS会将用户请求和自己已定义好的集群服务进行比对，如果用户请求的就是定义的集群服务，那么此时IPVS会强行修改数据包里的目标IP地址及端口，并将新的数据包发往POSTROUTING链。
4.  POSTROUTING链接收数据包后发现目标IP地址刚好是自己的后端服务器，那么此时通过选路，将数据包最终发送给后端的服务器。

LVS 由2部分程序组成，包括 ipvs 和 ipvsadm。

-   ipvs(ip virtual server)：一段代码工作在内核空间，叫ipvs，是真正生效实现调度的代码。
-   ipvsadm：另外一段是工作在用户空间，叫ipvsadm，负责为ipvs内核框架编写规则，定义谁是集群服务，而谁是后端真实的服务器(Real Server)。

Keepalived：

在lvs+keepalived环境里面，lvs主要的工作是提供调度算法，把客户端请求按照需求调度到RS，keepalived主要的工作是提供lvs控制器的一个冗余，并且对RS做健康检查，发现不健康的RS，就把它从lvs集群中剔除，RS 只负责提供服务。

#### LVS/NAT

通过网络地址转换，调度器重写请求报文的目标地址，根据预设的调度算法，将请求分派给后端的真实服务器；真实服务器的响应报文通过调度器时，报文的源地址被重写，再返回给客户，完成整个负载调度过程。

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648303811-image.png)

LVS/NAT

LVS-NAT模式下的系统结构图：

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648304377-image.png)

LVS-NAT模式下的系统结构图

#### LVS/DR

VS/DR通过改写请求报文的MAC地址，将请求发送到真实服务器，而真实服务器将响应直接返回给客户。同VS/TUN技术一样，VS/DR技术可极大地提高集群系统的伸缩性。这种方法没有IP隧道的开销，对集群中的真实服务器也没有必须支持**IP隧道**协议的要求，但是要求调度器与真实服务器都有一块网卡连在同一物理网段上。

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648304532-image.png)

LVS/DR

LVS/DR模式的流程大概如下：

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648305291-image.png)

LVS DR模式

1.  客户端发送请求至VIP，也就是访问服务，请求报文源地址是CIP，目标地址为VIP；
2.  LVS调度器接收到请求，报文在PREROUTING链检查，确定目的IP是本机，于是将报文发送至INPUT链，ipvs内核模块确定请求的服务是我们配置的LVS集群服务，然后根据用户设定的均衡策略选择某台后端RS，并将目标MAC地址修改RIP的MAC地址。因为调度器和后端服务器RS在同个网段，因此直接二层互通，将请求发给选择的RS处理；
3.  因为报文目的mac是本机，且RS上有配置VIP，因此RS能接收该报文。后端服务处理完请求后，将响应直接发往客户端，此时源IP地址为VIP，目标IP为CIP。

#### LVS/TUN

采用NAT技术时，由于请求和响应报文都必须经过调度器地址重写，当客户请求越来越多时，调度器的处理能力将成为瓶颈。为了解决这个问题，调度器把请求报文通过**IP隧道**转发至真实服务器，而真实服务器将响应直接返回给客户，所以调度器只处理请求报文。由于一般网络服务应答比请求报文大许多，采用 VS/TUN技术后，集群系统的最大吞吐量可以提高10倍。

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648304930-image.png)

LVS/TUN

LVS/TUN 的流程如下：

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648306573-image.png)

LVS/TUN 的流程

在原有的IP报文外再次封装多一层IP头部，内部IP头部(源地址为CIP，目标地址为VIP)，外层IP头部(源地址为DIP，目标IP为RIP)

1.  当用户请求到达DS，此时请求的数据报文会先到内核空间的PREROUTING链。 此时报文的源IP为CIP，目标IP为VIP 。
2.  PREROUTING检查发现数据包的目标IP是本机，将数据包送至INPUT链。
3.  IPVS比对数据包请求的服务是否为集群服务，若是，在请求报文的首部再次封装一层IP报文，封装源IP为DIP，目标IP为RIP。然后发至POSTROUTING链。 此时源IP为DIP，目标IP为RIP。
4.  POSTROUTING链根据最新封装的IP报文，将数据包发至RS（因为在外层封装多了一层IP首部，所以可以理解为此时通过隧道传输）。 此时源IP为DIP，目标IP为RIP。
5.  RS接收到报文后发现是自己的IP地址，就将报文接收下来，拆除掉最外层的IP后，会发现里面还有一层IP首部，而且目标是自己的lo接口VIP，那么此时RS开始处理此请求，处理完成之后，通过lo接口送给eth0网卡，然后向外传递。 此时的源IP地址为VIP，目标IP为CIP。
6.  响应报文最终送达至客户端。

特点：

-   DIP, VIP, RIP都应该是公网地址；
-   RS的网关不能也不可能指向DIP；
-   请求报文要经由Director，但响应不能经由Director；
-   此模式不支持端口映射；
-   RS的操作系统得支持隧道功能。

缺点:

由于后端服务器RS处理数据后响应发送给用户，此时需要租借大量IP（特别是后端服务器使用较多的情况下）。

优点:

LVS 调度器将TCP/IP请求进行重新封装并转发给后端服务器，由目标应用服务器直接回复用户。应用服务器之间是通过IP 隧道来进行转发，故两者可以存在于不同的网段中。

#### LVS三种模式的主要区别

<table><tbody><tr><td></td><td><strong>VS/NAT</strong></td><td><strong>VS/DR</strong></td><td><strong>VS/TUN</strong></td></tr><tr><td>通常支持节点数量</td><td>10 到 20 个，根据负载均衡器的处理能力而定</td><td>较高，可以支持 100 个服务节点</td><td>较高，可以支持 100 个服务节点</td></tr><tr><td>服务器网关</td><td>负载均衡器为服务器节点网关（load balancer）</td><td>服务节点同自己的网关或者路由器连接，不经过负载均衡器</td><td>服务器的节点同自己的网关或者路由器连接，不经过负载均衡器</td></tr><tr><td>对服务器的要求</td><td>服务节点可以使任何操作系统</td><td>服务器节点支持虚拟网卡设备，能够禁用设备的 ARP 响应</td><td>必须支持 IP 隧道，目前只有 Linux 系统支持</td></tr><tr><td>网络要求</td><td>拥有私有 IP 地址的局域网络（private）</td><td>拥有合法 IP 地址的局域，服务器节点与负载均衡器必须在同一个网段（LAN）</td><td>拥有合法 IP 地址的局域网或广域网（LAN/WAN）</td></tr><tr><td>服务节点安全性</td><td>较好，采用内部 IP，服务节点隐蔽</td><td>较差，采用公用 IP 地址，节点安全暴露</td><td>较差，采用公用 IP 地址，节点安全暴露</td></tr><tr><td>IP 要求</td><td>仅需要一个合法的 IP 地址作为 VIP 地址</td><td>除了 VIP 外，每个服务节点需拥有合法的 IP 地址，可以直接从路由到客户端</td><td>除了 VIP 外，每个服务节点需拥有合法的 IP 地址，可以直接从路由到客户端</td></tr><tr><td>特点</td><td>NAT</td><td>修改 MAC 地址</td><td>封装 IP</td></tr><tr><td>配置复杂度</td><td>简单</td><td>复杂</td><td>复杂</td></tr></tbody></table>

LVS三种模式的主要区别

### LVS 调度策略

-   无脑轮询，带权重的无脑轮询
-   最少链接，带权重的最少链接
-   IP\_Hash, IP\_HASH\_GROUP

LVS在内核中主要实现了十种调度算法：

#### 轮询调度

轮询调度（Round Robin 简称'RR'）算法就是按依次循环的方式将请求调度到不同的服务器上，该算法最大的特点就是实现简单。轮询算法假设所有的服务器处理请求的能力都一样的，调度器会将所有的请求平均分配给每个真实服务器。

#### 加权轮询调度

加权轮询（Weight Round Robin 简称'WRR'）算法主要是对轮询算法的一种优化与补充，LVS会考虑每台服务器的性能，并给每台服务器添加一个权值，如果服务器A的权值为1，服务器B的权值为2，则调度器调度到服务器B的请求会是服务器A的两倍。权值越高的服务器，处理的请求越多。

#### 最小连接调度

最小连接调度（Least Connections 简称'LC'）算法是把新的连接请求分配到当前连接数最小的服务器。最小连接调度是一种动态的调度算法，它**通过服务器当前活跃的连接数来估计服务器的情况**。调度器需要记录各个服务器已建立连接的数目，当一个请求被调度到某台服务器，其连接数加1；当连接中断或者超时，其连接数减1。

（集群系统的真实服务器具有相近的系统性能，采用最小连接调度算法可以比较好地均衡负载。)

#### 加权最小连接调度

加权最少连接（Weight Least Connections 简称'WLC'）算法是最小连接调度的超集，各个服务器相应的权值表示其处理性能。服务器的缺省权值为1，系统管理员可以动态地设置服务器的权值。加权最小连接调度在调度新连接时尽可能使服务器的已建立连接数和其权值成比例。调度器可以自动问询真实服务器的负载情况，并动态地调整其权值。

#### 基于局部的最少连接

基于局部的最少连接调度（Locality-Based Least Connections 简称'LBLC'）算法是针对请求报文的目标IP地址的负载均衡调度，目前主要用于Cache集群系统，因为在Cache集群客户请求报文的目标IP地址是变化的。这里假设任何后端服务器都可以处理任一请求，算法的设计目标是在服务器的负载基本平衡情况下，将相同目标IP地址的请求调度到同一台服务器，来提高各台服务器的访问局部性和Cache命中率，从而提升整个集群系统的处理能力。LBLC调度算法先根据请求的目标IP地址找出该目标IP地址最近使用的服务器，若该服务器是可用的且没有超载，将请求发送到该服务器；若服务器不存在，或者该服务器超载且有服务器处于一半的工作负载，则使用'最少连接'的原则选出一个可用的服务器，将请求发送到服务器。

#### 带复制的基于局部性的最少连接

带复制的基于局部性的最少连接（Locality-Based Least Connections with Replication 简称'LBLCR'）算法也是针对目标IP地址的负载均衡，目前主要用于Cache集群系统，它与LBLC算法不同之处是它要维护从一个目标IP地址到一组服务器的映射，而LBLC算法维护从一个目标IP地址到一台服务器的映射。按'最小连接'原则从该服务器组中选出一台服务器，若服务器没有超载，将请求发送到该服务器；若服务器超载，则按'最小连接'原则从整个集群中选出一台服务器，将该服务器加入到这个服务器组中，将请求发送到该服务器。同时，当该服务器组有一段时间没有被修改，将最忙的服务器从服务器组中删除，以降低复制的程度。

#### 目标地址散列调度

目标地址散列调度（Destination Hashing 简称'DH'）算法先根据请求的目标IP地址，作为散列键（Hash Key）从静态分配的散列表找出对应的服务器，若该服务器是可用的且并未超载，将请求发送到该服务器，否则返回空。

#### 源地址散列调度

源地址散列调度（Source Hashing 简称'SH'）算法先根据请求的源IP地址，作为散列键（Hash Key）从静态分配的散列表找出对应的服务器，若该服务器是可用的且并未超载，将请求发送到该服务器，否则返回空。它采用的散列函数与目标地址散列调度算法的相同，它的算法流程与目标地址散列调度算法的基本相似。

#### 最短的期望的延迟

最短的期望的延迟调度（Shortest Expected Delay 简称'SED'）算法基于WLC算法。举个例子吧，ABC三台服务器的权重分别为1、2、3 。那么如果使用WLC算法的话一个新请求进入时它可能会分给ABC中的任意一个。使用SED算法后会进行一个运算：

A：(1+1)/1=2、B:(1+2)/2=3/2、C:(1+3)/3=4/3，就把请求交给得出运算结果最小的服务器。

#### 最少队列调度

最少队列调度（Never Queue 简称'NQ'）算法，无需队列。如果有realserver的连接数等于0就直接分 配过去，不需要在进行SED运算。  

### LVS 强大的优势

-   IP 层的负载均衡协议，无应用层回调消耗。
-   通过LVS/DR或LVS/TUN模型的特性使得请求返回不过LVS。
-   自动故障转移，心跳检测。
-   配合主从 KeepAlive 加 VIP 实现自身高可用。

## Nginx 原理

Nginx是一个高性能的HTTP和反向代理服务器，也是一个IMAP/POP3/SMTP代理服务器。

Nginx是一款轻量级的Web服务器/反向代理服务器以及电子邮件代理服务器，并在一个BSD-like协议下发行。由俄罗斯的程序设计师 lgor Sysoev 所开发，供俄国大型的入口网站及搜索引擎Rambler使用。其特点是占有内存少，并发能力强，事实上nginx的并发能力确实在同类型的网页服务器中表现较好。

Nginx相较于Apache、lighttpd具有占有内存少，稳定性高等优势，并且依靠并发能力强，丰富的模块库以及友好灵活的配置而闻名。在Linux操作系统下，nginx 使用 epoll 事件模型，得益于此，nginx在 Linux 操作系统下效率相当高。同时 Nginx 在 OpenBSD 或 FreeBSD 操作系统上采用类似于 Epoll 的高效事件模型 kqueue。

### 反向代理和正向代理

#### 正向代理

正向代理是一个位于客户端和原始服务器之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。客户端必须要进行一些特别的设置才能使用正向代理。

正向代理类似一个跳板机，代理访问外部资源。**客户端必须设置正向代理服务器，当然前提是要知道正向代理服务器的IP地址，还有代理程序的端口。**

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648390773-image.png)

**正向代理**

**正向代理的用途：**

-   访问原来无法访问的资源，如 google。
-   可以做缓存，加速访问资源。
-   对客户端访问授权，上网进行认证。
-   代理可以记录用户访问记录（上网行为管理），对外隐藏用户信息。

#### 反向代理

反向代理（Reverse Proxy）实际运行方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。

反向代理中，**客户端是无感知代理的存在的，反向代理对外都是透明的**，访问者者并不知道自己访问的是一个代理。因为**客户端不需要任何配置就可以访问**。

nginx支持配置反向代理，通过反向代理实现网站的负载均衡。

**反向代理的作用：**

-   保证内网的安全，可以使用反向代理提供 WAF 功能，阻止web攻击。大型网站，通常将反向代理作为公网访问地址，Web 服务器是内网。
-   负载均衡，通过反向代理服务器来优化网站的负载。

#### 正向代理和反向代理的区别

从用途上来讲：

-   正向代理的典型用途是为在防火墙内的局域网客户端提供访问Internet的途径。正向代理还可以使用缓冲特性减少网络使用率。
-   反向代理的典型用途是将防火墙后面的服务器提供给Internet用户访问。反向代理还可以为后端的多台服务器提供负载平衡，或为后端较慢的服务器提供缓冲服务。另外，反向代理还可以启用高级URL策略和管理技术，从而使处于不同web服务器系统的web页面同时存在于同一个URL空间下。

从安全性来讲：

-   正向代理允许客户端通过它访问任意网站并且隐藏客户端自身（前提是确保代理服务器是安全可信赖的），因此你必须采取安全措施以确保仅为经过授权的客户端提供服务。
-   反向代理对外都是透明的，访问者并不知道自己访问的是一个代理。

### Nginx 职责分类

-   接入层 Nginx
-   应用层 Nginx

### Nginx 功能

-   请求解析
-   负载均衡：应用层的负载均衡
-   缓存调度：静态资源、动态请求的缓存
-   授权认证
-   接入处理：过滤非法请求
-   业务逻辑
-   相应处理
-   压缩技术

### 接入层 Nginx 的主要职责

接入层 Nginx 的主要职责的特点是业务无关。

-   请求解析
-   请求业务路由
-   业务负载均衡
-   响应压缩

### 应用层 Nginx 主要职责

-   应用负载均衡
-   缓存调度
-   授权认证
-   业务逻辑
-   业务限流
-   业务降级

### Nginx 的优势

-   作为Web服务器，Nginx处理静态文件、索引文件，自动索引的效率非常高。
-   作为代理服务器，Nginx可以实现无缓存的反向代理加速，提高网站运行速度。
-   作为负载均衡服务器，Nginx既可以在内部直接支持Rails和PHP，也可以支持HTTP代理服务器对外进行服务，同时还支持简单的容错和利用算法进行负载均衡。
-   在性能方面，Nginx是专门为性能优化而开发的，实现上非常注重效率。它采用内核Epoll模型，可以支持更多的并发连接，最大可以支持对5万个并发连接数的响应，而且只占用很低的内存资源。
-   在稳定性方面，Nginx采取了分阶段资源分配技术，使得CPU与内存的占用率非常低。Nginx官方表示，Nginx保持1万个没有活动的连接，而这些连接只占用2.5MB内存，因此，类似DOS这样的攻击对Nginx来说基本上是没有任何作用的。
-   在高可用性方面，Nginx支持热部署，启动速度特别迅速，因此可以在不间断服务的情况下，对软件版本或者配置进行升级，即使运行数月也无需重新启动，几乎可以做到7x24小时不间断地运行。

### Nginx 高性能原因

-   master-worker 进程模型
-   流式处理请求 workflow，有主子请求
-   协程机制
-   nginx lua
-   selector 模型 & epoll 模型

### 进程模型

Nginx 由一个master进程和多个worker进程组成，但master进程或者worker进程中并不会再创建线程。

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648311152-image.png)

Nginx 进程模型

#### master进程和worker进程的作用

master进程：

-   不需要处理网络事件，不负责业务的执行，只会通过管理worker等子进程来实现重启服务、平滑升级、更换日志文件、配置文件实时生效等功能。
-   master 是通过fork系统调用子进程来实现和子进程的通信。

worker进程：

-   用来处理master进程fork过来的请求。
-   worker进程是通过处理信号来实现和master通信的。

#### 信号的处理过程

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648311311-image.png)

Nginx 信号的处理过程

Master进程接收到信号是怎样进行处理的？

master进程接收到信号后，会先重新加载配置文件，然后再启动新的进程，并向所有老的进程发送信号，告诉他们可以光荣退休了。新的进程在启动后，就开始接受新的请求，而老的进程在收到来master信号后，就不再接收新的请求，并且在当前进程中的所有未处理完的请求处理完成后再退出。

Worker进程接收到信号是怎样进行处理的？

首先，worker进程之间是平等的，每个进程，处理请求的机会也是一样的。当我们提供80端口的http服务时，一个连接请求过来，每个进程都有可能处理这个连接，怎么做到的呢？首先，每个worker进程都是从master进程fork过来的，在master进程里面，先建立好需要listen的socket之后，然后再fork出多个worker进程，这样每个worker进程都可以去接受这个socket。一般来说，当一个连接进来后，所有在accept这个socket上面的进程都会收到通知，而只有一个进程可以接受这个连接，其他的则accept失败，这就是所谓的惊群现象。

那么为了解决这个问题，Nginx提供了一个accept\_mutex（可选项，默认打开）。这是一个加在accept上的一把共享锁。有了这把锁之后，同一时刻，就会只有一个进程在accept连接,这样就不会有惊群问题了。

当一个worker进程在accept这个连接之后，就开始读取请求，解析请求，处理请求，产生数据后，再返回给客户端，最后才断开连接。一个请求，完全由worker进程来处理，而且只在一个worker进程中处理。

高性能线程模型：

boss 进程 epoll 注册，由 worker 进程池竞争 accept mutex 用来获得连接的 accept 权限及后续的 recv, send 操作权限，本质在处理上仍然是等待阻塞式的效率。因此 woker 进程内的 workflow 不能有阻塞式的操作，解决方式是通过协程去处理。

协程与线程：

协程与线程的区别：

<table><tbody><tr><td></td><td>协程</td><td>线程</td></tr><tr><td>内存结构</td><td>栈，局部变量在用户空间模拟</td><td>栈，局部变量是内核空间的映射</td></tr><tr><td>关系</td><td>协程间是协作关系</td><td>多线程并发运行竞争 cpu</td></tr><tr><td>开销</td><td>切换开销小</td><td>切换开销大</td></tr><tr><td>加锁</td><td>临界区不需要加锁</td><td>临界区需要同步加锁</td></tr><tr><td>阻塞时策略</td><td>遇到阻塞主动放弃切换</td><td>遇到阻塞进入等待 cpu 切换</td></tr></tbody></table>

协程与线程的区别

### Nginx工作原理

Nginx 会按需同时运行多个进程：一个主进程和几个工作进程，配置了缓存时还会有缓存加载器进程（cache loader）和缓存管理器进程（cache manager)等。所有进程均是仅含有一个线程，并主要通过“共享内存”的机制实现进程间通信。主进程以root用户身份运行，而 worker、cache loader 和 cache manager 均应以非特权用户身份运行。

#### HTTP请求和响应的过程

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648313707-image.png)

HTTP请求和响应的过程

当它接收到一个 HTTP 请求时，它仅仅是通过查找配置文件将此次请求映射到一个 location block，而此 location 中配置的各个指令则会启动不同的模块去完成工作，因此模块可以看做Nginx 真正的劳动工作者。通常一个 location 中的指令会涉及一个 handler 模块和多个 filter 模块（当然，多个location可以用同一个模块）。handler 模块负责处理请求，完成响应内容的生成，而 filter 模块对响应内容进行处理。

#### HTTP 反向代理服务器

由于 Nginx 具有“强悍”的高并发高负载能力，因此一般会作为前端的服务器直接向客户端提供静态文件服务。但也有一些复杂、多变的业务不适合放到 Nginx 服务器上，这时会用 Apache、Tomcat 等服务器来处理。于是，Nginx 通常会被配置为既是静态 Web 服务器也是反向代理服务器，不适合 Nginx 处理的请求就会直接转发到上游服务器中处理。

Nginx作为HTTP服务器以及反向代理服务器：

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648313958-image.png)

HTTP 反向代理服务器

#### 反向代理服务器转发请求的流程

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648314013-image.png)

反向代理服务器转发请求的流程

Nginx减轻了上游服务器的并发压力；延长了一个请求的处理时间，并增加了用于缓存请求内容的内存和磁盘空间。

### Nginx 应用

#### 负载均衡

nginx 配置：

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648390273-image.png)

_负载均衡 nginx 配置_

动静分离：

动态资源代理到后端服务器集群，静态资源则由 Nginx 处理。

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648389129-image.png)

动静分离 Nginx 配置

#### 代理缓存

```nginx
# 申明一个 cache 缓存节点的路径 proxy_cache_path /usr/local/opt/openresty/nginx/tmp_cache levels=1:2 keys_zone=tmp_cache:100m inactive=7d max_size=100g; # /usr/local/opt/openresty/nginx/tmp_cache，把缓存文件放在哪里 # levels：目录设置两层结构用来缓存 # keys_Zone：指定了一个叫 tmp_Cache 的缓存区，并且设置了 100m 的内存用来存储缓存 key 到文件路径的位置 # inactive：缓存文件超过 7 天后自动释放淘汰 maxsize：缓存文件总大小超过 100g 后自动释放淘汰 Server{ listen 7777; # 当用户访问 resources 目录下的文件即认为其在访问静态资源 location /resources/{ alias //usr/local/opt/openresty/nginx/html/; } location / { proxy_pass http://backend_server; proxy_cache tmp_cache; proxy_cache_valid 200 206 304 302 10d; proxy_cache_key $uri; }
```

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
