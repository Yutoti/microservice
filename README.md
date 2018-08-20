# microservice
对微服务的理解

因为实习的部门是基于微服务架构进行软件开发的，自己了解了一些微服务的知识，总结在这里。
****
微服务是一种应用架构方式，它将一个大的应用按照服务细分为多个“微服务”，每个微服务都有自己独立的功能，由独立的开发团队开发部署，其使用的技术栈不限。关于微服务我主要关注两个大的方面，一是它与先前的架构方式有什么区别，或者说软件架构方式的演变；第二是微服务本身。

# 一 软件架构方式的演变

单体式架构最终演变成分布式架构中间还有其它解决方案：

单体式架构->应用服务和数据服务分离->使用缓存改善性能->使用应用服务器&数据库集群->数据库读写分离->分布式应用&数据库->分布式架构。

而分布式架构也有两种解决方案：SOA架构和微服务架构。

这里主要比较单体式架构、SOA与微服务的区别。

## 1.&nbsp;单体式架构 vs 微服务

单体式架构较为简单，但随着需求的增加，代码日积月累，整个工程的维护变得极其困难，同时稳定性、扩展性也不好。

微服务架构则将一个大的应用分为多个小的微服务，各个服务功能独立，由独立的团队开发部署，可以使用不同的开发语言。由此带来的好处是：

1. 易于开发和维护本团队微服务的功能；
2. 每个微服务高内聚低耦合，具有更高的灵活性；
3. 每个微服务功能独立，缩短开发周期；
4. 技术栈不受限。

但微服务并非是完美的，其缺点在于：

1. 各个微服务需要协调完成整体功能，服务联调困难，运维成本高；
2. 数据冗余、代码重复；
3. 网络拥塞和延迟；

## 2.&nbsp;SOA vs 微服务

SOA是面向服务的架构方式，与微服务概念类似，两者最大的差别在于，微服务不限技术栈，可以使用多种语言分别编写。

# 二 微服务本身

关于微服务本身，主要关注三个方面：
1. 客户端与微服务如何通信；
2. 微服务之间如何通信；
3. 服务之间如何发现。

## 1.&nbsp;客户端与微服务如何通信
（1）最简单的方式：直接通信

理论上说，每个微服务都有一个对外的URL，当从客户端发来请求时，请求可以直接传送到相应的微服务的服务器上，但这种方式有几个弊端：

1. 客户端的一个操作可能涉及许多微服务，如果采用直接通信的方式，客户端需要分别给各个微服务发送请求，效率太低；
2. 各个微服务通信采用的可能是不同的协议，并非Web友好的，只适合内部使用；
3. 直接通信的方式使得开发人员很难重构微服务（如将一个微服务拆分成两个）。

综上，我们不采用这种方式，而采用API Gateway的方式。

（2）采用API Gateway服务器

API Gateway本质上是一个服务器，也是进入系统的唯一节点，这跟面向对象的外观模式很像。API Gateway封装了内部系统架构，并提供统一的API给客户端，为系统提供请求转发、合成和协议转换的功能。

具体来说就是API Gateway暴露一个粗粒度的API给客户端，客户端通过调用该API向API GateWay服务器发送请求，API Gateway接着将请求路由到各自的微服务，调用多个微服务并聚合各个微服务的结果，最后将该客户端请求的所有结果返回。

有一些开源的API Gateway可以使用，如Netflix zuul、apiaxle等，但这些开源项目使用依赖具体的语言，且都不具备聚合服务的功能。通常各个企业都有自己实现的API Gateway。

## 2.&nbsp;微服务之间如何通信

通常来说，每个微服务都在不同的进程中（可以部署在同一台机器上，也可以在不同机器上），各个微服务之间的交互需要通过进程间通信（IPC）来实现。

IPC有两种方式：

1. 异步的基于消息通信的方式；
2. 同步的基于请求/响应的方式。

### 异步的基于消息通信的方式（Kafka）

当使用该方式时，客户端通过向服务端发送消息提交请求，服务端如果需要回复，则发送另一个独立的消息给客户端。在这种方式下，通信是异步的，客户端不会因为等待而阻塞。

消息通过channel发送，有两类channel：点对点（一对一）和发布/订阅（一对多）。

有许多基于异步消息的消息系统可以选择，如**Kafka**、RabbitMQ、ActiveMQ等。

消息格式有两类：文本和二进制。文本格式的例子包括JSON和XML。

### 同步的基于请求/响应的方式（Spring Boot;Thrift, Dubbo）

1. 基于HTTP协议的REST方式（Spring Boot）
2. 基于TCP协议的RPC方式（Thrift, Dubbo）

1.&nbsp;基于HTTP协议的REST方式

REST架构将网络上的一切看做“资源”，每个资源都有唯一的标识符URL，通过HTTP定义的几个动词（GET、POST等）来操作资源。REST基于HTTP协议，采用的是同步的请求/响应模式。

2.&nbsp;基于TCP协议的RPC方式。

基于RPC方式，可以通过参数传递的方式在本地像调用本地方法（服务）一样调用远程服务器上的方法（服务），RPC会隐藏底层的通信细节，不需要直接处理Socket通信或HTTP通信；RPC也是同步的基于请求/响应模型。

**REST和RPC的对比**

REST基于HTTP，更通用，支持各个语言，开发更灵活；
RPC基于TCP，传输协议更高效，更安全可控。

### 同步和异步的对比

同步的方式比较简单，一致性强，但调用容易出现问题，效率不如异步高；

异步的方式在分布式系统应用比较广泛，它能减少服务间的耦合，还能成为服务调用之间的缓冲；能保证服务的性能。

## 3.&nbsp;服务之间如何发现

在微服务架构中，服务实例的网络位置都是动态分配的，而且因为扩展、失效和升级等需求，服务实例会经常动态改变，因此，客户端代码需要使用一种更加复杂的服务发现机制。

服务间发现主要有两种方式：

 1. 客户端发现；
 2. 服务端发现。
 
### 客户端发现模式
当使用客户端发现模式时，客户端负责决定相应服务实例的网络位置，并且对请求实现负载均衡。客户端从一个服务注册服务中查询，其中是所有可用服务实例的库。客户端使用负载均衡算法从多个服务实例中选择出一个，然后发出请求。

### 服务端发现模式

客户端通过负载均衡器向某个服务提出请求，负载均衡器向服务注册表发出请求，将每个请求转发往可用的服务实例。跟客户端发现一样，服务实例在服务注册表中注册或者注销。客户端无需关注发现的逻辑，由服务端发现请求的微服务。
