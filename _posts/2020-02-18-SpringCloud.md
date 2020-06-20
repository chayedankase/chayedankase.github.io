---
layout: post
title: springcloud全家桶
categories: [微服务]
description: springcloud各个组件
keywords: spring
---



# SpringCloud

## 一、微服务基础知识

### 1.1 系统架构的演变

随着互联网的发展，网站应用的规模不断扩大，常规的应用架构已无法应对，分布式服务架构以及微服 务架构势在必行，亟需一个治理系统确保架构有条不紊的演进。

#### 1.1.1 单体应用架构

Web应用程序发展的早期，大部分web工程(包含前端页面,web层代码,service层代码,dao层代码)是将所有的功能模块,打包到一起并放在一个web容器中运行。

![image-20200218115913882]({{site.url}}\imgs\springcloud\image-20200218115913882.png)

比如搭建一个电商系统：客户下订单，商品展示，用户管理。这种将所有功能都部署在一个web容器中 运行的系统就叫做单体架构。

**优点：**

- 所有的功能集成在一个项目工程中
- 项目架构简单，前期开发成本低，周期短，小型项目的首选。

**缺点：**

- 全部功能集成在一个工程中，对于大型项目不易开发、扩展及维护。
- 系统性能扩展只能通过扩展集群结点，成本高、有瓶颈。
- 技术栈受限。

#### 1.1.2 垂直应用架构

当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，将应用拆成互不相干的几个应用，以提升效率。

![image-20200218120604122]({{site.url}}\imgs\springcloud\image-20200218120604122.png)

**优点**：

- 项目架构简单，前期开发成本低，周期短，小型项目的首选。
- 通过垂直拆分，原来的单体项目不至于无限扩大
- 不同的项目可采用不同的技术。

**缺点**：

- 全部功能集成在一个工程中，对于大型项目不易开发、扩展及维护。
- 系统性能扩展只能通过扩展集群结点，成本高、有瓶颈。

#### 1.1.3 分布式SOA架构

##### 1.1.3.1 什么是SOA

SOA全称为 Service-Oriented Architecture，即面向服务的架构。它可以根据需求通过网络对松散耦合 的粗粒度应用组件(服务)进行分布式部署、组合和使用。一个服务通常以独立的形式存在于操作系统进 程中。

站在功能的角度，把业务逻辑抽象成可复用、可组装的服务，通过服务的编排实现业务的快速再生，目的：把原先固有的业务功能转变为通用的业务服务，实现业务逻辑的快速复用。

通过上面的描述可以发现 SOA有如下几个特点：**分布式、可重用、扩展灵活、松耦合**

##### 1.1.3.2 SOA架构

当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。

![image-20200218121017845]({{site.url}}\imgs\springcloud\image-20200218121017845.png)

**优点**：

- 抽取公共的功能为服务,提高开发效率
- 对不同的服务进行集群化部署解决系统压力
- 基于esb/dubbo减少系统耦合

**缺点**：

- 抽取服务的粒度较大
- 服务提供方与调用方接口耦合度较高

#### 1.1.4 微服务架构

![image-20200218122104867]({{site.url}}\imgs\springcloud\image-20200218122104867.png)

**优点:**

- 通过服务的原子化拆分，以及微服务的独立打包、部署和升级，小团队的交付周期将缩短，运维成本也将大幅度下降
- 微服务遵循单一原则。微服务之间采用Restful等轻量协议传输。

**缺点:**

- 微服务过多，服务治理成本高，不利于系统维护。
- 分布式系统开发的技术成本高（容错、分布式事务等）。

#### 1.1.5  SOA与微服务的关系

**SOA**（ Service Oriented Architecture）“面向服务的架构”:他是一种设计方法，其中包含多个服务， 服务之间通过相互依赖最终提供一系列的功能。一个服务 通常以独立的形式存在与操作系统进程 中。各个服务之间 通过网络调用。

**微服务架构**  其实和 SOA架构类似,微服务是在 SOA上做的升华，微服务架构强调的一个重点是“业务需 要彻底的组件化和服务化”，原有的单个业务系统会拆分为多个可以独立开发、设计、运行的小应用。 这些小应用之间通过服务完成交互和集成。

| **功能** | **SOA**              | **微服务**                   |
| :------- | :------------------- | ---------------------------- |
| 组件大小 | 大块业务逻辑         | 单独任务或小块业务逻辑       |
| 耦合     | 通常松耦合           | 总是松耦合                   |
| 公司架构 | 任何类型             | 小型、专注于功能交叉团队     |
| 管理     | 着重中央管理         | 着重分散管理                 |
| 目标     | 确保应用能够交互操作 | 执行新功能、快速拓展开发团队 |

### 1.2 分布式核心知识

#### 1.2.1 分布式中的远程调用

在微服务架构中，通常存在多个服务之间的远程调用的需求。远程调用通常包含两个部分：序列化和通信协议。常见的序列化协议包括json、xml、hession、protobuf、thrift、text、bytes等，目前主流的 远程调用技术有基于HTTP的Restful接口以及基于TCP的RPC协议。

##### 1.2.1.1 Restful接口

 REST，即Representational State Transfer的缩写，如果一个架构符合REST原则，就称它为Restful架构。

**资源（Resources）**

所谓"资源"，就是网络上的一个实体，或者说是网络上的一个具体信息。它可以是一段文本、一张图片、一首歌曲、一种服务，总之就是一个具体的实在。你可以用一个URI（统一资源定位符）指向它，每种资源对应一个特定的URI。要获取这个资源，访问它的URI就可以，因此URI就成了每一个资源的地 址或独一无二的识别符。REST的名称"表现层状态转化"中，省略了主语。"表现层"其实指的是"资源"（Resources）的"表现层"。

**表现层（Representation）**

"资源"是一种信息实体，它可以有多种外在表现形式。我们把"资源"具体呈现出来的形式，叫做它的"表 现层"（Representation）。比如，文本可以用txt格式表现，也可以用HTML格式、XML格式、JSON格式表现，甚至可以采用二进制格式；图片可以用JPG格式表现，也可以用PNG格式表现。URI只代表资源的实体，不代表它的形式。严格地说，有些网址最后的".html"后缀名是不必要的，因为这个后缀名表示 格式，属于"表现层"范畴，而URI应该只代表"资源"的位置。

**状态转化（State Transfer）**

访问一个网站，就代表了客户端和服务器的一个互动过程。在这个过程中，势必涉及到数据和状态的变 化。互联网通信协议HTTP协议，是一个无状态协议。这意味着，所有的状态都保存在服务器端。因此，如果客户端想要操作服务器，必须通过某种手段，让服务器端发生"状态转化"（State Transfer）。客户端用到的手段，只能是HTTP协议。具体来说，就是HTTP协议里面，四个表示操作方式的动词：GET、POST、PUT、DELETE。它们分别对应四种基本操作：GET用来获取资源，POST用来新建资源（也可以用于更新资源），PUT用来更新资源，DELETE用来删除资源。

综合上面的解释，我们总结一下什么是**Restful**架构：

- 每一个URI代表一种资源；
- 客户端和服务器之间，传递这种资源的某种表现层；
- 客户端通过四个HTTP动词，对服务器端资源进行操作，实现"表现层状态转化"。

##### 1.2.1.2 RPC协议

**RPC**（Remote Procedure Call） 一种进程间通信方式。允许像调用本地服务一样调用远程服务。RPC 框架的主要目标就是让远程服务调用更简单、透明。RPC框架负责屏蔽底层的传输方式（TCP或者UDP）、序列化方式（XML/JSON/二进制）和通信细节。开发人员在使用的时候只需要了解谁在什么位置提供了什么样的远程服务接口即可，并不需要关心底层通信细节和调用过程。

![image-20200218123358109]({{site.url}}\imgs\springcloud\image-20200218123358109.png)

##### 1.2.1.3 区别与联系

| **比较项** | **Restful** | **RPC**     |
| ---------- | ----------- | ----------- |
| 通讯协议   | HTTP        | 一般使用TCP |
| 性能       | 略低        | 较高        |
| 灵活度     | 高          | 低          |
| 应用       | 微服务架构  | SOA架构     |

1、HTTP相对更规范，更标准，更通用，无论哪种语言都支持http协议。如果你是对外开放API，例如开放平台，外部的编程语言多种多样，你无法拒绝对每种语言的支持，现在开源中间件，基本最先支持的几个协议都包含Restful。

2、 RPC框架作为架构微服务化的基础组件，它能大大降低架构微服务化的成本，提高调用方与服务提供方的研发效率，屏蔽跨进程调用函数（服务）的各类复杂细节。让调用方感觉就像调用本地函数一样 调用远端函数、让服务提供方感觉就像实现一个本地函数一样来实现服务。

#### 1.2.2 分布式中的CAP原理

现如今，对于多数大型互联网应用，分布式系统（distributed system）正变得越来越重要。分布式系统的最大难点，就是各个节点的状态如何同步。CAP定理是这方面的基本定理，也是理解分布式系统的 起点。

CAP理论由 Eric Brewer在ACM研讨会上提出，而后CAP被奉为分布式领域的重要理论。分布式系统的 CAP理论，首先把分布式系统中的三个特性进行了如下归纳：

![image-20200218124045598]({{site.url}}\imgs\springcloud\image-20200218124045598.png)

**Consistency（一致性）**：数据一致更新，所有数据的变化都是同步的

**Availability（可用性）**：在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求 

**Partition tolerance（分区容忍性）**：某个节点的故障，并不影响整个系统的运行

> 通过学习CAP理论，我们得知任何分布式系统只可同时满足二点，没法三者兼顾，既然一个分布
>
> 式系统无法同时满足一致性、可用性、分区容错性三个特点，所以我们就需要抛弃一样：



![image-20200218124312524]({{site.url}}\imgs\springcloud\image-20200218124312524.png)

| 选择 | **说 明**                                                    |
| ---- | ------------------------------------------------------------ |
| CA   | 放弃分区容错性，加强一致性和可用性，其实就是传统的关系型数据库的选择 |
| AP   | 放弃一致性（这里说的一致性是强一致性），追求分区容错性和可用性，这是很多分布式 系统设计时的选择，例如很多NoSQL系统就是如此 |
| CP   | 放弃可用性，追求一致性和分区容错性，基本不会选择，网络问题会直接让整个系统不可 用 |

需要明确一点的是，在一个分布式系统当中，分区容忍性和可用性是最基本的需求，所以在分布是系统 中，我们的系统最当关注的就是A（可用性）P（容忍性），通过补偿的机制寻求数据的一致性

### 1.3常见微服务框架

#### 1.3.1 SpringCloud

![image-20200218124736710]({{site.url}}\imgs\springcloud\image-20200218124736710.png)

SpringCloud是一系列框架的有序集合。它利用SpringBoot的开发便利性巧妙地简化了分布式系统基 础设施的开发，如**服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控**等，都可以用Spring Boot的开发风格做到一键启动和部署。Spring Cloud并没有重复制造轮子，它只是将目前各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过Spring Boot风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包。

#### 1.3.2 ServiceComb

![image-20200218124934898]({{site.url}}\imgs\springcloud\image-20200218124934898.png)

Apache ServiceComb是业界第一个Apache微服务顶级项目， 是一个开源微服务解决方案,致力于帮助企业、用户和开发者将企业应用轻松微服务化上云，并实现对微服务应用的高效运维管理。其提供一站 式开源微服务解决方案，融合SDK框架级、0侵入ServiceMesh场景并支持多语言。

#### 1.3.3 ZeroC ICE

![image-20200218125118672]({{site.url}}\imgs\springcloud\image-20200218125118672.png)

ZeroC IceGrid是ZeroC公司的杰作，继承了CORBA的血统，是新一代的面向对象的分布式系统中间件。作为一种微服务架构，它基于RPC框架发展而来，具有良好的性能与分布式能力。

## 二、SpringCloud概述

### 2.1微服务中的相关概念

#### 2.1.1服务注册与发现

**服务注册**：服务实例将自身服务信息注册到注册中心。这部分服务信息包括服务所在主机IP和提供服务的Port，以及暴露服务自身状态以及访问协议等信息。

**服务发现**：服务实例请求注册中心获取所依赖服务信息。服务实例通过注册中心，获取到注册到其中的 服务实例的信息，通过这些信息去请求它们提供的服务。

![image-20200218125651596]({{site.url}}\imgs\springcloud\image-20200218125651596.png)

#### 2.1.2 负载均衡

负载均衡是高可用网络基础架构的关键组件，通常用于将工作负载分布到多个服务器来提高网站、应用、数据库或其他服务的性能和可靠性。

![image-20200218125832211]({{site.url}}\imgs\springcloud\image-20200218125832211.png)

#### 2.1.3 熔断

**熔断**这一概念来源于电子工程中的断路器（Circuit Breaker）。在互联网系统中，当下游服务因访问压力过大而响应变慢或失败，上游服务为了保护系统整体的可用性，可以暂时切断对下游服务的调用。这 种牺牲局部，保全整体的措施就叫做熔断。

![image-20200218130002926]({{site.url}}\imgs\springcloud\image-20200218130002926.png)

#### 2.1.4 链路追踪

随着微服务架构的流行，服务按照不同的维度进行拆分，一次请求往往需要涉及到多个服务。互联网应用构建在不同的软件模块集上，这些软件模块，有可能是由不同的团队开发、可能使用不同的编程语言来实现、有可能布在了几千台服务器，横跨多个不同的数据中心。因此，就需要对一次请求涉及的多个 服务链路进行日志记录，性能监控即**链路追踪**

![image-20200218130118844]({{site.url}}\imgs\springcloud\image-20200218130118844.png)

#### 2.1.5 API网关

随着微服务的不断增多，不同的微服务一般会有不同的网络地址，而外部客户端可能需要调用多个服务 的接口才能完成一个业务需求，如果让客户端直接与各个微服务通信可能出现：

- 客户端需要调用不同的url地址，增加难度
- 再一定的场景下，存在跨域请求的问题
- 每个微服务都需要进行单独的身份认证

针对这些问题，API网关顺势而生。

**API网关**直面意思是将所有API调用统一接入到API网关层，由网关层统一接入和输出。一个网关的基本功能有：统一接入、安全防护、协议适配、流量管控、长短链接支持、容错能力。有了网关之后，各个 API服务提供团队可以专注于自己的的业务逻辑处理，而API网关更专注于安全、流量、路由等问题。

![image-20200218130350989]({{site.url}}\imgs\springcloud\image-20200218130350989.png)

### 2.2 SpringCloud的架构

#### 2.2.1 SpringCloud中的核心组件

**Spring Cloud**的本质是在 SpringBoot的基础上，增加了一堆微服务相关的规范，并对应用上下文（Application Context）进行了功能增强。既然 SpringCloud是规范，那么就需要去实现，目前Spring Cloud规范已有 Spring官方，Spring Cloud Netﬂix，Spring Cloud Alibaba等实现。通过组件化的方式，Spring Cloud将这些实现整合到一起构成全家桶式的微服务技术栈。

**Spring Cloud Netﬂix组件**

| **组件名称** | **作用**       |
| ------------ | -------------- |
| Eureka       | 服务注册中心   |
| Ribbon       | 客户端负载均衡 |
| Feign        | 声明式服务调用 |
| Hystrix      | 客户端容错保护 |
| Zuul         | API服务网关    |

**Spring Cloud Alibaba组件**

| **组件名称** | **作用**       |
| ------------ | -------------- |
| Nacos        | 服务注册中心   |
| Sentinel     | 客户端容错保护 |

**Spring Cloud原生及其他组件**

| **组件**      | **作用**       |
| ------------- | -------------- |
| Consul        | 服务注册中心   |
| Conﬁg         | 分布式配置中心 |
| Gateway       | API服务网关    |
| Sleuth/Zipkin | 分布式链路追踪 |

#### 2.2.2 SpringCloud的体系结构

![image-20200218131211534]({{site.url}}\imgs\springcloud\image-20200218131211534.png)

从上图可以看出Spring Cloud各个组件相互配合，合作支持了一套完整的微服务架构。

- **注册中心**  负责服务的注册与发现，很好将各服务连接起来
- **断路器**  负责监控服务之间的调用情况，连续多次失败进行熔断保护。
- **API网关**  负责转发所有对外的请求和服务
- **配置中心**  提供了统一的配置信息管理服务,可以实时的通知各个服务获取最新的配置信息
- **链路追踪技术**  可以将所有的请求数据记录下来，方便我们进行后续分析

各个组件又提供了功能完善的**dashboard监控平台**,可以方便的监控各组件的运行状况

## 三、案例准备

使用微服务架构的分布式系统,微服务之间通过网络通信。我们通过服务提供者与服务消费者来描述微服
务间的调用关系。为了后续技术的学习，我们先进行案例准备，完成基础的远程调用。再在后续的学习中用学到的组件不断的完善案例，以体现各个组件的作用。

我们以电商系统中常见的用户下单为例，用户向订单微服务发起一个购买的请求。在进行保存订单之前
需要调用商品微服务查询当前商品库存，单价等信息。在这种场景下，订单微服务就是一个服务消费
者，商品微服务就是一个服务提供者

![image-20200219110352451]({{site.url}}\imgs\springcloud\image-20200219110352451.png)

### 3.1 创建父项目

创建一个maven的父项目，用于管理依赖版本和公共依赖。

1. 创建一个maven项目,Springcloud-demo。

2. 导入依赖

   ```XML
   <parent>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-parent</artifactId>
       <version>2.1.6.RELEASE</version>
   </parent>
   
   <dependencies>
    	<!-- 依赖一个lombok方便写代码 -->   
       <dependency>
           <groupId>com.develhack</groupId>
           <artifactId>develhacked-lombok</artifactId>
           <version>0.1.5</version>
           <scope>provided</scope>
       </dependency>
   </dependencies>
   
   <dependencyManagement>
       <dependencies>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-dependencies</artifactId>
               <version>Greenwich.SR5</version>
               <type>pom</type>
               <scope>import</scope>
           </dependency>
       </dependencies>
   </dependencyManagement>
   ```

### 3.2 创建服务提供者

1. 创建商品服务 product-server。

2. 添加依赖。

   ```xml-dtd
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <parent>
           <artifactId>springcloud-demo</artifactId>
           <groupId>org.example</groupId>
           <version>1.0-SNAPSHOT</version>
       </parent>
       <modelVersion>4.0.0</modelVersion>
   
       <artifactId>product-server</artifactId>
   
       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
       </dependencies>
   </project>
   ```

3. 编写启动类和controller服务。

   ```java
   /**
    * 产品服务启动类
    *
    * @author Xue-Pan
    * @date 2020/2/17
    */
   @SpringBootApplication
   public class ProductServerApplication {
       public static void main(String[] args) {
           SpringApplication.run(ProductServerApplication.class, args);
       }
   }
   
   /**
    * 产品controller
    *
    * @author Xue-Pan
    * @date 2020/2/17
    */
   @RestController
   @RequestMapping("prodect")
   public class ProductController {
   
       @GetMapping("/{id}")
       public Product getPrdect(@PathVariable Integer id){
           Product product = null;
           if(id == 1){
               product = new Product();
               product.setName("测试商品");
               product.setPrice("50元");
               product.setId(1);
           }
           return product;
       }
   }
   
   /**
    * 产品实体类
    *
    * @author Xue-Pan
    * @date 2020/2/17
    */
   @Data
   @ToString
   public class Product {
   
       private Integer id;
       private String name;
       private String price;
   }
   ```

4. 编写配置文件

   ```yml
   server:
     port: 8001
   spring:
     application:
       name: product-server
   ```

### 3.3 创建服务消费者

1. 创建订单服务 order-client

2. 添加依赖。

   ```xml-dtd
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <parent>
           <artifactId>springcloud-demo</artifactId>
           <groupId>org.example</groupId>
           <version>1.0-SNAPSHOT</version>
       </parent>
       <modelVersion>4.0.0</modelVersion>
   
       <artifactId>order-client</artifactId>
   
       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
       </dependencies>
   
   </project>
   
   ```

3. 编写启动类、服务调用类、产品实体类、配置类。

   ```java
   /**
    * 订单启动类
    *
    * @author Xue-Pan
    * @date 2020/2/18
    */
   @SpringBootApplication
   public class OrderApplication {
       public static void main(String[] args) {
           SpringApplication.run(OrderApplication.class, args);
       }
   }
   
   /**
    * 订单控制器
    *
    * @author Xue-Pan
    * @date 2020/2/18
    */
   @RestController
   @RequestMapping("order")
   public class OrderController {
       
   }
   
   /**
    * 产品实体类
    *
    * @author Xue-Pan
    * @date 2020/2/17
    */
   @Data
   @ToString
   public class Product {
   
       private Integer id;
       private String name;
       private String price;
   }
   
   /**
    * 配置类
    *
    * @author Xue-Pan
    * @date 2020/2/18
    */
   @Configuration
   public class OrderConfig {
   
   }
   ```

4. 编写配置文件。

   ```YML
   server:
     port: 8002
   spring:
     application:
       name: order-client
   ```

### 3.4 远程服务调用

前文已经编写了三个基础的微服务，在用户下单时需要调用商品微服务获取商品数据。那应该怎么做呢？总人皆知商品微服务提供了供人调用的HTTP接口。所以可以再下定单的时候使用http请求的相关 工具类完成，如常见的HttpClient，OkHttp，当然也可以使用Spring提供的RestTemplate

#### 3.4.1 RestTemplate介绍

Spring框架提供的RestTemplate类可用于在应用中调用rest服务，它简化了与http服务的通信方式，统
一了Restful的标准，封装了http链接，我们只需要传入url及返回值类型即可。相较于之前常用的
HttpClient，RestTemplate是一种更优雅的调用Restful服务的方式。

在Spring应用程序中访问第三方REST服务与使用Spring RestTemplate类有关。RestTemplate类的设计
原则与许多其他Spring 模板类(例如JdbcTemplate、JmsTemplate)相同，为执行复杂任务提供了一种具
有默认行为的简化方法。

RestTemplate默认依赖JDK提供http连接的能力（HttpURLConnection），如果有需要的话也可以通过
setRequestFactory方法替换为例如Apache HttpComponents、Netty或OkHttp等其它HTTP library。
考虑到RestTemplate类是为调用REST服务而设计的，因此它的主要方法与REST的基础紧密相连就不足
为奇了，后者是HTTP协议的方法:HEAD、GET、POST、PUT、DELETE和OPTIONS。例如，
RestTemplate类具有headForHeaders()、getForObject()、postForObject()、put()和delete()等方法

#### 3.4.2 RestTemplate方法介绍

该模板类的主要切入点为以下几个方法（对应着HTTP的六个主要方法）：

| HTTP methods | RestTemplate methods                                         |
| ------------ | ------------------------------------------------------------ |
| GET          | getForObect(String,Class<T>,Object...)          getForEntity(String,Class<T>,Object...) |
| POST         | postForLocation(String,Object,Object...)     postForObject(String,Object,Class<T>,Object...) |
| PUT          | put(String,Object,Object...)                                 |
| DELETE       | delete(String,Object...)                                     |
| HEAD         | headForHeaders(String,Object...)                             |
| OPTIONS      | optionsForAllow(String,Object...)                            |
| any          | exchange(String,HttpMethod,HttpEntity<?>,Class<T>,Object...)         execute(String,HttpMethod,RequestCallback,ResponseExtractor<T>,Object...) |

#### 3.4.3 通过RestTemplate调用微服务

1. 在order-client的配置类OrderConfig中注入 RestTemplate。

   ```java
   @Bean
   public RestTemplate restTemplate(){
       return new RestTemplate();
   }
   ```

2. 在OrderController中编写订单查询方法，在查询方法中调用商品服务。

   ```java
   @Autowired
   private RestTemplate restTemplate;
   
   @GetMapping("/{id:\\d+}")
   public Object getOrder(@PathVariable Integer id) {
       return restTemplate.getForObject("http://127.0.0.1:8001/product/"+id, Product.class);
   }
   ```

3. 启动服务提供者和服务消费者并访问 http://localhost:8002/order/1

   ​	![image-20200219130931549]({{site.url}}\imgs\springcloud\image-20200219130931549.png)

#### 3.4.4 硬编码问题

至此已经可以通过RestTemplate调用商品微服务的RestFul API接口。但是我们把提供者的网络地址
（ip，端口）等硬编码到了代码中，这种做法存在许多问题：

- 应用场景有局限
- 无法动态调整

那么应该怎么解决呢，SpringCloud已经为我们提供了解决方案：**服务注册和服务发现**

## 四、服务注册与发现

### 4.1 微服务的注册中心

注册中心可以说是微服务架构中的”通讯录“，它记录了服务和服务地址的映射关系。在分布式架构中，
服务会注册到这里，当服务需要调用其它服务时，就这里找到服务的地址，进行调用。

![image-20200219131542646]({{site.url}}\imgs\springcloud\image-20200219131542646.png)

#### 4.1.1 注册中心的主要作用

服务注册中心（下称注册中心）是微服务架构非常重要的一个组件，在微服务架构里主要起到了协调者的作用。注册中心一般包含如下几个功能：

1. 服务发现：

   - 服务注册/反注册：保存服务提供者和服务调用者的信息

   - 服务订阅/取消订阅：服务调用者订阅服务提供者的信息，最好有实时推送的功能 

   - 服务路由（可选）：具有筛选整合服务提供者的能力。

2. 服务配置：

   - 配置订阅：服务提供者和服务调用者订阅微服务相关的配置

   - 配置下发：主动将配置推送给服务提供者和服务调用者

3. 服务健康监测：

   - 检测服务提供者的健康情况

#### 4.1.2 常用注册中心

**Zookeeper**

zookeeper它是一个分布式服务框架，是Apache Hadoop的一个子项目，它主要是用来解决分布式应用中经常遇到的一些数据管理问题，如：统一命名服务、状态同步服务、集群管理、分布式应用配置项的管理等。简单来说zookeeper=文件系统+监听通知机制。

 **Eureka**

Eureka是在Java语言上，基于Restful API开发的服务注册与发现组件，Springcloud Netﬂix中的重要组件。

**Consul**

Consul是由HashiCorp基于Go语言开发的支持多数据中心分布式高可用的服务发布和注册服务软件， 采用Raft算法保证服务的一致性，且支持健康检查。

**Nacos**

Nacos是一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。简单来说 Nacos就是注册中心 +配置中心的组合，提供简单易用的特性集，帮助我们解决微服务开发必会涉及到的服务注册 与发现，服务配置，服务管理等问题。Nacos还是 Spring Cloud Alibaba组件之一，负责**服务注册与发现**。

最后我们通过一张表格大致了解Eureka、Consul、Zookeeper的异同点。选择什么类型的服务注册与发现组件可以根据自身项目要求决定。

| 组件名    | 语言 | CAP  | 一致性算法 | 服务健康检查 | 对外暴露接口 |
| --------- | ---- | ---- | ---------- | ------------ | ------------ |
| Eureka    | Java | AP   | 无         | 可配支持     | HTTP         |
| Consul    | Go   | CP   | Raft       | 支持         | HTTP/DNS     |
| Zookeeper | Java | CP   | Paxos      | 支持         | 客户端       |
| Nacos     | Java | AP   | Raft       | 支持         | HTTP         |

### 4.2 Eureka

#### 4.2.1 Eureka基础

##### 4.2.1.1 Eureka基础概述

Eureka是Netﬂix开发的服务发现框架，SpringCloud将它集成在自己的子项目spring-cloud-netﬂix中，
实现SpringCloud的服务发现功能。

![image-20200219140938358]({{site.url}}\imgs\springcloud\image-20200219140938358.png)

上图简要描述了Eureka的基本架构，由3个角色组成：

1、Eureka Server

提供服务注册和发现

2、Service Provider

服务提供方   将自身服务注册到Eureka，从而使服务消费方能够找到

3、Service Consumer

服务消费方   从Eureka获取注册服务列表，从而能够消费服务

##### 4.2.1.2 Eureka的交互流程与原理

![image-20200219141334478]({{site.url}}\imgs\springcloud\image-20200219141334478.png)

图是来自Eureka官方的架构图，大致描述了Eureka集群的工作过程。图中包含的组件非常多，可能比 较难以理解，我们用通俗易懂的语言解释一下：

Application Service相当于服务提供者，Application Client相当于服务消费者； Make Remote Call，可以简单理解为调用Restful API；

us-east-1c、us-east-1d等都是zone，它们都属于us-east-1这个region；

**由图可知,Eureka包含两个组件：Eureka Server和 Eureka Client，它们的作用如下：**

Eureka Client是一个Java客户端，用于简化与Eureka Server的交互；

Eureka Server提供服务发现的能力，各个微服务启动时，会通过Eureka Client向Eureka Server 进行注册自己的信息（例如网络信息），Eureka Server会存储该服务的信息；

微服务启动后，会周期性地向Eureka Server发送心跳（默认周期为30秒）以续约自己的信息。如果Eureka Server在一定时间内没有接收到某个微服务节点的心跳，Eureka Server将会注销该微服务节点（默认90秒）；

每个Eureka Server同时也是Eureka Client，多个Eureka Server之间通过复制的方式完成服务注册表的同步；

Eureka Client会缓存Eureka Server中的信息。即使所有的Eureka Server节点都宕掉，服务消费 者依然可以使用缓存中的信息找到服务提供者。

综上，Eureka通过心跳检测、健康检查和客户端缓存等机制，提高了系统的灵活性、可伸缩性和可用 性。

#### 4.2.2 搭建Eureka服务

Eureka是用java语言写的，因而搭建过程与我们创建java项目一致。

##### 4.2.2.1 创建项目

1. 创建模块 eureka-server。

   在springcloud-demo项目下创建子模块 eureka-server。

2. 导入依赖。

   > 导入依赖时注意低版本的Springcloud eureka的依赖为 spring-cloud-starter-eureka-server
   >
   > 高版本的变更为 spring-cloud-starter-netflix-eureka-server

   ```XML
   <parent>
       <artifactId>springcloud-demo</artifactId>
       <groupId>org.example</groupId>
       <version>1.0-SNAPSHOT</version>
   </parent>
   <modelVersion>4.0.0</modelVersion>
   
   <artifactId>eureka-server</artifactId>
   
   <dependencies>
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
       </dependency>
   </dependencies>
   ```

3. 编写配置文件。

   ```YML
   server:
     port: 9000
   eureka:
     instance:
       hostname: localhost
     client:
       # 服务端自己不注册
       register-with-eureka: false
       # 服务端自己不获取注册信息
       fetch-registry: false
       service-url:
         #客户端与eureka服务交互的地址
         defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
   ```

4. 编写启动类并启动。

   ```java
   /**
    * eureka服务启动类
    *
    * @author Xue-Pan
    * @date 2020/2/17
    */
   @SpringBootApplication
   //引入eurekaServer,表示此为eureka服务端
   @EnableEurekaServer
   public class EurekaServerApplication {
       public static void main(String[] args) {
           SpringApplication.run(EurekaServerApplication.class, args);
       }
   }
   ```

5. 打开eureka后台管理页面。

   输入 http://localhost:9000打开eureka后台管理页面。

   ![image-20200219144734070]({{site.url}}\imgs\springcloud\image-20200219144734070.png)

##### 4.2.2.2 注册服务

将商品服务注册到eureka中，以便后续订单服务来调用。

1. 在product-server模块中引入eureka依赖。

   ```XML
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   ```

2. 添加eureka的配置。

   ```YML
   eureka:
     client:
       service-url:
         #设置连接的eureka服务地址
         defaultZone: http://localhost:9000/eureka
     instance:
       #设置链结后的id显示信息为 ip:服务名称:端口号
       instance-id: ${spring.cloud.client.ip-address}:${spring.application.name}:${server.port}
       #设置显示 实例id的ip信息
       prefer-ip-address: true
   ```

3. 在启动类上增加引入eureka的注解。

   从Spring Cloud Edgware版本开始，@EnableDiscoveryClient或 @EnableEurekaClient省略。只需加上相关依赖，并进行相应配置，即可将微服务注册到服务发现组件上。

   ```java
   /**
    * 产品服务启动类
    *
    * @author Xue-Pan
    * @date 2020/2/17
    */
   @SpringBootApplication
   @EnableEurekaClient
   //@EnableDiscoveryClient 或者这个注解
   public class ProductServerApplication {
       public static void main(String[] args) {
           SpringApplication.run(ProductServerApplication.class, args);
       }
   }
   ```

#### 4.2.3 Eureka 的自我保护

微服务第一次注册成功之后，每30秒会发送一次心跳将服务的实例信息注册到注册中心。通知EurekaServer 该实例仍然存在。如果超过90秒没有发送更新，则服务器将从注册信息中将此服务移除。

Eureka Server在运行期间，会统计心跳失败的比例在15分钟之内是否低于85%，如果出现低于的情况（在单机调试的时候很容易满足，实际在生产环境上通常是由于网络不稳定导致），Eureka Server会将当前的实例注册信息保护起来，同时提示这个警告。保护模式主要用于一组客户端和Eureka Server之间存在网络分区场景下的保护。一旦进入保护模式，Eureka Server将会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据（也就是不会注销任何微服务）

验证完自我保护机制开启后，并不会马上呈现到web上，而是默认需等待5 分钟（可以通过
eureka.server.wait-time-in-ms-when-sync-empty 配置），即5 分钟后你会看到下面的提示信息：![image-20200219205042912]({{site.url}}\imgs\springcloud\image-20200219205042912.png)

如何关闭自我保护

通过设置 `eureka.enableSelfPreservation=false`来关闭自我保护功能。==生产模式此功能不建议关闭==

#### 4.2.4 Eureka元数据

Eureka的元数据有两种：标准元数据和自定义元数据。

标准元数据：主机名、IP地址、端口号、状态页和健康检查等信息，这些信息都会被发布在服务注 册表中，用于服务之间的调用。

自定义元数据：可以使用eureka.instance.metadata-map配置，符合KEY/VALUE的存储格式。这些元数据可以在远程客户端中访问。

在程序中可以使用DiscoveryClient获取指定微服务的所有元数据信息，Eureka已经在自动配置中帮我们注入了此对象，可以直接使用。

#### 4.2.5 使用Eureka进行远程调用

1. 在order-client子模块中引入eureka的依赖。

   ```XML
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   ```

2. 添加eureka的配置。

   ```YML
   eureka:
     instance:
       hostname: localhost
     client:
       service-url:
         defaultZone: http://127.0.0.1:9000/eureka
   ```

3. 修改启动类，添加引入eureka的注解（高版本此步骤可省略）。

4. 修改OrderController,利用元数据获取地址。

   ```java
   /**
    * 订单控制器
    *
    * @author Xue-Pan
    * @date 2020/2/18
    */
   @RestController
   @RequestMapping("order")
   public class OrderController {
     @Autowired
     private RestTemplate restTemplate;
   
     //Eureka元数据
     @Autowired
     private DiscoveryClient discoveryClient;
   
     @GetMapping("/{id:\\d+}")
     public Object getOrder(@PathVariable Integer id) {
     //通过元数据和服务应用名称获取服务实例列表（同一个服务应用可以注册多个成为集群，所以这里有可能是多个，后续会有响应的内容）
     List<ServiceInstance> instances = discoveryClient.getInstances("product-server");
     ServiceInstance serviceInstance = instances.get(0);
     //通过元数据即可获取到地址、端口。即只要有服务应用名称，即可确定要调用的服务地址。
     //这里可以思考一下,如果获取到了多个服务实例serviceInstance,任意一个实例都可以用来进行接口的调用，如何选择、选择的算法就是一种负载均衡。 
     String host = serviceInstance.getHost();
     int port = serviceInstance.getPort();
     return restTemplate.getForObject("http://"+host + ":" + port+"/product/"+id, Product.class);
       }
   }
   ```

5. 启动服务进行远程调用测试。

   启动eureka-server、product-server、order-client，然后访问 http://127.0.0.1:8002/order/1

   ![image-20200219213651720]({{site.url}}\imgs\springcloud\image-20200219213651720.png)

   调用成功。再看注册中心的控制台 http://127.0.0.1:9000

   ![image-20200219214004960]({{site.url}}\imgs\springcloud\image-20200219214004960.png)

   如上图可以看到商品服务和订单服务都被注册了，其中商品服务配置了 instance-id，所以在status处显示了ip。订单服务未配置，所以显示了默认信息，不利于区分。

#### 4.2.6 Eureka高可用集群

前面实现了单节点的Eureka Server的服务注册与服务发现功能。Eureka Client会定时连接Eureka Server，获取注册表中的信息并缓存到本地。微服务在消费远程API时总是使用本地缓存中的数据。因此一般来说，即使Eureka Server发生宕机，也不会影响到服务之间的调用。但如果EurekaServer宕机时，某些微服务也出现了不可用的情况，Eureka Server中的缓存若不被刷新，就可能会影响到微服务的调用，甚至影响到整个应用系统的高可用。因此，在生成环境中，通常会部署一个高可用的Eureka Server集群。

Eureka Server可以通过运行多个实例并相互注册的方式实现高可用部署，Eureka Server实例会彼此增量地同步信息，从而确保所有节点数据一致。事实上，节点之间相互注册是Eureka Server的默认行为。

##### 4.2.6.1 搭建Eureka集群

1. 修改本地host文件。

   此处使用hostname是因为eureka只是部署在一台机器上，如果是真正的分布式，则可以直接使用IP

   ```
   127.0.0.1 eureka1
   127.0.0.1 eureka2
   ```

2. 修改eureka-server的配置文件。

   ```YML
   spring:
     application:
       name: eureka-server
   ---
   server:
     port: 9000
   eureka:
     instance:
     # 此处没有配置host文件，因而不使用hostname,直接使用本地ip127.0.0.1
       hostname: eureka1
     client:
       # 服务端自己不注册
       #register-with-eureka: false
       # 服务端自己不获取注册信息
       #fetch-registry: false
       service-url:
         #客户端与eureka服务交互的地址
         #defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
         defaultZone: http://eureka2:9001/eureka/
     #server:
       #设置关闭自我保护功能，生产上线时一定要开启
       #enable-self-preservation: false
   spring:
     profiles: eureka1
   ---
   server:
     port: 9001
   eureka:
     instance:
       hostname: eureka2
     client:
       # 服务端自己不注册
       #register-with-eureka: false
       # 服务端自己不获取注册信息
       #fetch-registry: false
       service-url:
         #客户端与eureka服务交互的地址
         #defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
         defaultZone: http://eureka1:9000/eureka/
       #server:
       #设置关闭自我保护功能，生产上线时一定要开启
       #enable-self-preservation: false
   spring:
     profiles: eureka2
   
   ```

3. 在idea上新增一个启动入口，与原有的启动入口分别使用不同的profile来启动。

   分别使用profile: eureka1、eureka2来启动

   ![image-20200219225719740]({{site.url}}\imgs\springcloud\image-20200219225719740.png)

4. 分别查看启动的eureka-server管理页面。

   ![image-20200219230011333]({{site.url}}\imgs\springcloud\image-20200219230011333.png)

   如上图显示，表示集群搭建成功。

   如果registered-replicas显示正确，available-replicas中的内如显示到unavailable-replicas中，则表示集群通过配置service-url正常通讯，但在通过hostname,或者ip进行副本是否可用的确认时，对比失败造成的。即配置service-url时使用了ip,而又未配置prefer-ip-address: true。[建议参考博客](https://www.cnblogs.com/lonelyJay/p/9940199.html)

5. 将服务注册到eureka集群。

   修改服务提供者和消费者的配置文件即可

   ```YML
   client:
       service-url:
         defaultZone: http://eureka1:9000/eureka,http://eureka2:9001/eureka
   ```

   将集群的节点都配置上，防止某一节点挂掉。

#### 4.2.7 Eureka常见问题

##### 4.2.7.1服务注册慢

默认情况下，服务注册到Eureka Server的过程较慢。[SpringCloud](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.1.2.RELEASE/single/spring-cloud-netflix.html#_why_is_it_so_slow_to_register_a_service)官方文档中给出了详细的原因

![image-20200220124421246]({{site.url}}\imgs\springcloud\image-20200220124421246.png)

大致含义：服务的注册涉及到心跳，默认心跳间隔为30s。在实例、服务器、客户端都在本地缓存中具有相同的元数据之前，服务不可用于客户端发现（所以可能需要3次心跳）。可以通过配置`eureka.instance.leaseRenewalIntervalInSeconds`(心跳频率)加快客户端连接到其他服务的过程。在生产中，最好坚持使用默认值，因为在服务器内部有一些计算，他们对续约做出假设。

##### 4.2.7.2 服务节点剔除

默认情况下，由于Eureka Server剔除失效服务间隔时间为90s且存在自我保护的机制。所以不能有效而 迅速的剔除失效节点，这对开发或测试会造成困扰。解决方案如下：

**Eureka Server：**

配置关闭自我保护，设置剔除无效节点的时间间隔

```yml
eureka:
  instance:
    hostname: eureka2
  client:
    # 服务端自己不注册默认为true
    #register-with-eureka: false
    # 服务端自己不获取注册信息 默认为true
    #fetch-registry: false
    service-url:
      #客户端与eureka服务交互的地址
      #defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
      defaultZone: http://eureka1:9000/eureka/
  server:
    #设置关闭自我保护功能，生产上线时一定要开启
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 4000 #剔除时间间隔,单位:毫秒
```

**Eureka Client：**

//配置开启健康检查，并设置续约时间

设置心跳检测时间。

```YML
eureka:
  instance:
    hostname: localhost
    lease-expiration-duration-in-seconds: 10 #eureka client发送心跳给server端后，续 约到期时间（默认90秒）
    lease-renewal-interval-in-seconds: 5 #发送心跳续约间隔
  client:
    service-url:
      defaultZone: http://eureka1:9000/eureka,http://eureka2:9001/eureka
```

##### 4.2.7.3 监控页面ip显示

在Eureka Server的管控台中，显示的服务实例名称默认情况下是微服务定义的名称和端口。为了更好 的对所有服务进行定位，微服务注册到Eureka Server的时候可以手动配置示例ID。配置方式如下

```YML
eureka:
  instance:
	instance-id: ${spring.cloud.client.ip-address}:${server.port}
	#spring.cloud.client.ip-address 获取到ip
```

![image-20200220130902876]({{site.url}}\imgs\springcloud\image-20200220130902876.png)

#### 4.2.7 Eureka源码解析

##### 4.2.7.1 服务注册源码解析

###### 4.2.7.1.1 EnableEurekaServer注解作用

通过前面我们知道注册中心的搭建在引入依赖后，需要在启动类上添加@EnableEurekaServer注解，而正是这个注解激活EurekaServer的自动配置。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(EurekaServerMarkerConfiguration.class)
public @interface EnableEurekaServer {
}
```

通过@Import向容器中注入了配置类 EurekaServerMarkerConfiguration。

```java
@Configuration
public class EurekaServerMarkerConfiguration {

	@Bean
	public Marker eurekaServerMarkerBean() {
		return new Marker();
	}
	class Marker {
	}
}
```

而配置类又向容器中注入了一个marker标记类，它会通过条件注解 @ConditionalOnBean来控制自动配置类的注入。

###### 4.2.7.1.2 EurekaServerAutoConfiguration自动配置类

SpringCloud对EurekaServer的封装使得发布一个EurekaServer无比简单，根据自动装载原则可以在

spring-cloud-netflix-eureka-server-2.1.5.RELEASE.jar下找到 spring.factories

![image-20200220132619941]({{site.url}}\imgs\springcloud\image-20200220132619941.png)

spring容器启动时，会加载EurekaServerAutoConfiguration类，实例化它的对象。

```java
@Configuration
@Import(EurekaServerInitializerConfiguration.class)
//在这里就对Marker对象进行了判断，只有容器中有了这个对象才会进行自动配置。
@ConditionalOnBean(EurekaServerMarkerConfiguration.Marker.class)
@EnableConfigurationProperties({ EurekaDashboardProperties.class,
		InstanceRegistryProperties.class })
@PropertySource("classpath:/eureka/server.properties")
public class EurekaServerAutoConfiguration extends WebMvcConfigurerAdapter {
    ....
}
```

现在我们展开来说这个Eureka服务端的自动配置类;

1. 这个配置类实例化的前提条件是上下文中存在EurekaServerMarkerConﬁguration.Marker 这个
   bean,解释了上面的问题。
2. 通过@EnableConﬁgurationProperties({ EurekaDashboardProperties.class,InstanceRegistryProperties.class })导入了两个配置类。
3. EurekaDashboardProperties ：配置EurekaServer的管控台。
4. InstanceRegistryProperties ：配置期望续约数量和默认的通信数量。
5. 通过@Import({EurekaServerInitializerConﬁguration.class})引入启动配置类。

源码暂时不写了，有时间仔细看。。。。

### 4.3 Consul

在Euraka的GitHub上，宣布Eureka 2.x闭源。近这意味着如果开发者继续使用作为2.x 分支上现有工作repo 一部分发布的代码库和工件，则将自负风险。springcloud从未使用过2.X，但未雨绸缪，对于注册中心我们还有很多可替换方案。

#### 4.3.1 Consul入门

##### 4.3.1.1 Consul概述

![image-20200220151241722]({{site.url}}\imgs\springcloud\image-20200220151241722.png)

Consul是 HashiCorp公司推出的开源工具，用于实现分布式系统的服务发现与配置。与其它分布式服务注册与发现的方案，Consul的方案更“一站式”，内置了服务注册与发现框 架、分布一致性协议实现、健康检查、Key/Value存储、多数据中心方案，不再需要依赖其它工具（比如 ZooKeeper等）。使用起来也较 为简单。Consul使用 Go语言编写，因此具有天然可移植性(支持Linux、windows和 Mac OS X)；安装包仅包含一个可执行文件，方便部署，与 Docker等轻量级容器可无缝配合。

**Console的优势：**

- 使用 Raft算法来保证一致性,比复杂的 Paxos算法更直接.相比较而言, zookeeper采用的是 Paxos,而 etcd使用的则是 Raft。
- 支持多数据中心，内外网的服务采用不同的端口进行监听。 多数据中心集群可以避免单数据中心
- 的单点故障,而其部署则需要考虑网络延迟,分片等情况等。 zookeeper和 etcd均不提供多数据中 心功能的支持。
- 支持健康检查。 etcd不提供此功能。
- 支持 http和 dns协议接口。 zookeeper的集成较为复杂, etcd只支持 http协议。
- 官方提供 web管理界面, etcd无此功能。
- 综合比较, Consul作为服务注册和配置管理的新星,比较值得关注和研究。

**特性：**

- 服务发现
- 健康检查
- Key/Value存储
- 多数据中心

##### 4.3.1.2 Consul与Eureka的区别

**（1）一致性**

Consul强一致性（CP）

- 服务注册相比Eureka会稍慢一些。因为Consul的raft协议要求必须过半数的节点都写入成功才认 为注册成功
- Leader挂掉时，重新选举期间整个consul不可用。保证了强一致性但牺牲了可用性。

Eureka保证高可用和最终一致性（AP）

- 服务注册相对要快，因为不需要等注册信息replicate到其他节点，也不保证注册信息是否 replicate成功
- 当数据出现不一致时，虽然A, B上的注册信息不完全相同，但每个Eureka节点依然能够正常对外提供服务，这会出现查询服务信息时如果请求A查不到，但请求B就能查到。如此保证了可用性但牺 牲了一致性。

**（2）开发语言和使用**

- eureka就是个servlet程序，跑在servlet容器中
- Consul则是go编写而成，安装启动即可

##### 4.3.1.3 Consul下载与安装

官网：https://www.consul.io

[下载页面](https://www.consul.io/downloads.html)，可以选择windows、linux等各个操作系统的安装包。

![image-20200220154931641]({{site.url}}\imgs\springcloud\image-20200220154931641.png)

**Linux：**

```SHELL
##从官网下载最新版本的Consul服务
wget https://releases.hashicorp.com/consul/1.5.3/consul_1.5.3_linux_amd64.zip ##使用unzip命令解压
unzip consul_1.5.3_linux_amd64.zip
##将解压好的consul可执行命令拷贝到/usr/local/bin目录下
cp consul /usr/local/bin
##测试一下
consul
```

开发模式启动

```SHELL
##已开发者模式快速启动，-client指定客户端可以访问的ip地址
[root@node01 ~]# consul agent -dev -client=0.0.0.0
==> Starting Consul agent...
Version: 'v1.5.3'
Node ID: '49ed9aa0-380b-3772-a0b6-b0c6ad561dc5' Node name: 'node01'
Datacenter: 'dc1' (Segment: '<all>')
Server: true (Bootstrap: false)
Client Addr: [127.0.0.1] (HTTP: 8500, HTTPS: -1, gRPC: 8502, DNS: 8600) Cluster Addr: 127.0.0.1 (LAN: 8301, WAN: 8302)
Encrypt: Gossip: false, TLS-Outgoing: false, TLS-Incoming: false,
Auto-Encrypt-TLS: false
```

**Windows：**

下载解压后出现 consul.exe,进入cmd命令窗口执行 `consul agent -dev -client=0.0.0.0`

![image-20200220155750442]({{site.url}}\imgs\springcloud\image-20200220155750442.png)

实现开发者模式启动。

#### 4.3.2 基于Consul的服务注册

复制product-server项目，变更文件夹及模块名称为consul-product-server。

![image-20200220160833882]({{site.url}}\imgs\springcloud\image-20200220160833882.png) 

修改pom.xml，更新依赖，并将consul-product-server加入spring-cloud-demo的子模块中。

```XML
<parent>
        <artifactId>springcloud-demo</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
<modelVersion>4.0.0</modelVersion>

<artifactId>consul-product-server</artifactId>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--SpringCloud提供的基于Consul的服务发现-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-consul-discovery</artifactId>
    </dependency>
    <!--actuator用于心跳检查-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>

------spring-cloud-demo pom.xml
<modules>
     <module>eureka-server</module>
     <module>product-server</module>
     <module>consul-product-server</module>
     <module>order-client</module>
</modules>
```

修改配置文件。

```YML
server:
  port: 8061
spring:
  application:
    name: consul-product-server
  cloud:
    consul:
      #consul服务的地址
      host: 127.0.0.1
      #consul服务的端口
      port: 8500
      discovery:
        # 是否注册，默认是true
        register: true
        # 实例id
        instance-id: ${spring.application.name}-1
        #服务实例名称
        service-name: ${spring.application.name}
        #服务实例端口
        port: ${server.port}
        #健康检查路径
        health-check-path: /actuator/health
        #健康检查时间间隔
        health-check-interval: 15s
        #开启ip地址注册
        prefer-ip-address: true
        #实例的请求ip
        ip-address: ${spring.cloud.client.ip-address}
```

其中spring.cloud.consul 中添加consul的相关配置

- host：表示Consul的Server的请求地址
- port：表示Consul的Server的端口
- discovery：服务注册与发现的相关配置
  - nstance-id ：实例的唯一id（推荐必填），spring cloud官网文档的推荐，为了保证生成一
    个唯一的id ，也可以换成
    `${spring.application.name}:${spring.cloud.client.ipAddress}`
  - prefer-ip-address：开启ip地址注册
  - ip-address：当前微服务的请求ip

访问http://127.0.0.1:8500

![image-20200220163846168]({{site.url}}\imgs\springcloud\image-20200220163846168.png)

#### 4.3.3 基于Consul的服务发现

复制一份order-client模块，变更文件夹名及模块名为 consul-order-client,启动类改为ConsulOrderApplication。

![image-20200220164419996]({{site.url}}\imgs\springcloud\image-20200220164419996.png)

修改pom.xml，更新依赖，并将consul-order-server加入spring-cloud-demo的子模块中。

```XML
<parent>
    <artifactId>springcloud-demo</artifactId>
    <groupId>org.example</groupId>
    <version>1.0-SNAPSHOT</version>
</parent>
<modelVersion>4.0.0</modelVersion>

<artifactId>consul-order-client</artifactId>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--SpringCloud提供的基于Consul的服务发现-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-consul-discovery</artifactId>
    </dependency>
    <!--actuator用于心跳检查-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>

----- spring-cloud-demo
 <modules>
     <module>eureka-server</module>
     <module>product-server</module>
     <module>consul-product-server</module>
     <module>order-client</module>
     <module>consul-order-client</module>
</modules>
```

修改配置文件

```YML
server:
  port: 8062
spring:
  application:
    name: consul-order-server
  cloud:
    consul:
      #consul服务的地址
      host: 127.0.0.1
      #consul服务的端口
      port: 8500
      discovery:
        # 是否注册，默认是true
        register: true
        # 实例id
        instance-id: ${spring.application.name}-1
        #服务实例名称
        service-name: ${spring.application.name}
        #服务实例端口
        port: ${server.port}
        #健康检查路径
        health-check-path: /actuator/health
        #健康检查时间间隔
        health-check-interval: 15s
        #开启ip地址注册
        prefer-ip-address: true
        #实例的请求ip
        ip-address: ${spring.cloud.client.ip-address}
```

依旧使用DiscoveryClient来进行远程调用，只是修改服务调用时获取服务实例的服务名称

```java
@RestController
@RequestMapping("order")
public class OrderController {
    @Autowired
    private RestTemplate restTemplate;

    @Autowired
    private DiscoveryClient discoveryClient;

    @GetMapping("/{id:\\d+}")
    public Object getOrder(@PathVariable Integer id) {
        //这里是获取consul-product-server 实例
        List<ServiceInstance> instances = discoveryClient.getInstances("consul-product-server");
        ServiceInstance serviceInstance = instances.get(0);
        String host = serviceInstance.getHost();
        int port = serviceInstance.getPort();
        return restTemplate.getForObject("http://"+host + ":" + port+"/product/"+id, Product.class);
    }
}
```

访问consul管理页面http://127.0.0.1:8500,可以看到consul-order-client也已经注册上去了。

![image-20200220171704458]({{site.url}}\imgs\springcloud\image-20200220171704458.png)

在访问一下http://127.0.0.1:8062/order/1 查询id为1的订单,访问成功。

![image-20200220171742747]({{site.url}}\imgs\springcloud\image-20200220171742747.png)

#### 4.3.4 Consul高可用集群

![image-20200220174009105]({{site.url}}\imgs\springcloud\image-20200220174009105.png)

此图是官网提供的一个事例系统图，图中的Server是consul服务端高可用集群，Client是consul客户端。consul客户端不保存数据，客户端将接收到的请求转发给响应的Server端。Server之间通过局域网 或广域网通信实现数据一致性。每个Server或Client都是一个consul agent。Consul集群间使用了 GOSSIP协议通信和raft一致性算法。上面这张图涉及到了很多术语：

- Agent——agent是一直运行在Consul集群中每个成员上的守护进程。通过运行 consul agent来启动。agent可以运行在client或者server模式。指定节点作为client或者server是非常简单的，除非有其 他agent实例。所有的agent都能运行DNS或者HTTP接口，并负责运行时检查和保持服务同步。
- Client—— 一个Client是一个转发所有RPC到server的代理。这个client是相对无状态的。client唯 一执行的后台活动是加入LAN
- gossip池。这有一个最低的资源开销并且仅消耗少量的网络带宽。
- Server——一个server是一个有一组扩展功能的代理，这些功能包括参与Raft选举，维护集群状 态，响应RPC查询，与其他数据中心交互WANgossip和转发查询给leader或者远程数据中心。
- DataCenter——虽然数据中心的定义是显而易见的，但是有一些细微的细节必须考虑。例如，在EC2中，多个可用区域被认为组成一个数据中心？我们定义数据中心为一个私有的，低延迟和高带宽的一个网络环境。这不包括访问公共网络，但是对于我们而言，同一个EC2中的多个可用区域可以被认为是一个数据中心的一部分。
- Consensus——在我们的文档中，我们使用Consensus来表明就leader选举和事务的顺序达成一 致。由于这些事务都被应用到有限状态机上，Consensus暗示复制状态机的一致性。
- Gossip——Consul建立在Serf的基础之上，它提供了一个用于多播目的的完整的gossip协议。Serf提供成员关系，故障检测和事件广播。更多的信息在gossip文档中描述。这足以知道gossip使用基于UDP的随机的点到点通信。
- LAN Gossip——它包含所有位于同一个局域网或者数据中心的所有节点。 
- WAN Gossip——它只包含Server。这些server主要分布在不同的数据中心并且通常通过因特网或者广 域网通信。

在每个数据中心，client和server是混合的。一般建议有3-5台server。这是基于有故障情况下的可用性 和性能之间的权衡结果，因为越多的机器加入达成共识越慢。然而，并不限制client的数量，它们可以 很容易的扩展到数千或者数万台。

同一个数据中心的所有节点都必须加入gossip协议。这意味着gossip协议包含一个给定数据中心的所有 节点。这服务于几个目的：第一，不需要在client上配置server地址。发现都是自动完成的。第二，检 测节点故障的工作不是放在server上，而是分布式的。这是的故障检测相比心跳机制有更高的可扩展性。第三：它用来作为一个消息层来通知事件，比如leader选举发生时。

每个数据中心的server都是Raft节点集合的一部分。这意味着它们一起工作并选出一个leader，一个有额外工作的server。leader负责处理所有的查询和事务。作为一致性协议的一部分，事务也必须被复制到所有其他的节点。因为这一要求，当一个非leader得server收到一个RPC请求时，它将请求转发给集 群leader。

server节点也作为WAN gossip Pool的一部分。这个Pool不同于LAN Pool，因为它是为了优化互联网更 高的延迟，并且它只包含其他Consul server节点。这个Pool的目的是为了允许数据中心能够以low- touch的方式发现彼此。这使得一个新的数据中心可以很容易的加入现存的WAN gossip。因为server都运行在这个pool中，它也支持跨数据中心请求。当一个server收到来自另一个数据中心的请求时，它随 即转发给正确数据中想一个server。该server再转发给本地leader。

这使得数据中心之间只有一个很低的耦合，但是由于故障检测，连接缓存和复用，跨数据中心的请求都 是相对快速和可靠的。

集群后续略，有空再继续整理。。。。



### 4.4 Nacos

#### 4.4.1 Nacos基础

##### 4.4.1.1 Nacos概述

Nacos是阿里的一个开源产品，它是针对微服务架构中的服务发现、配置管理、服务治理的综合型解决方案。在这里我们首先了解Nacos的服务发现与服务治理。

Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。

Nacos 帮助您更敏捷和容易地构建、交付和管理微服务平台。 Nacos 是构建以“服务”为中心的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施。

##### 4.4.1.2 Nacos特性

Nacos主要提供以下四大功能：

1. **服务发现与服务健康检查** 

   Nacos使服务更容易注册，并通过DNS或HTTP接口发现其他服务，Nacos还提供服务的实时健康检查，以防 止向不健康的主机或服务实例发送请求。

2.  **动态配置管理** 

   动态配置服务允许您在所有环境中以集中和动态的方式管理所有服务的配置。Nacos消除了在更新配置时重新 部署应用程序，这使配置的更改更加高效和灵活。

3. **动态DNS服务** 

   Nacos提供基于DNS 协议的服务发现能力，旨在支持异构语言的服务发现，支持将注册在Nacos上的服务以 域名的方式暴露端点，让三方应用方便的查阅及发现。

4. **服务和元数据管理** 

   Nacos 能让您从微服务平台建设的视角管理数据中心的所有服务及元数据，包括管理服务的描述、生命周 期、服务的静态依赖分析、服务的健康状态、服务的流量管理、路由及安全策略

#### 4.4.2 Nacos服务搭建

nacos也是java语言开发的，它的服务搭建有2种方式，一种是下载源码自己编译启动，一种是直接下载编译好的压缩包，这里我们直接使用编译好的压缩包。下载地址 https://github.com/alibaba/nacos/releases

##### 4.4.2.1 Nacos服务下载与启动

进入下载页面，我们选择最新版的nacos服务。

![1586151657935]({{site.url}}\imgs\springcloud\1586151657935.png)

下载并解压，得到如下目录。

![1586151719701]({{site.url}}\imgs\springcloud\1586151719701.png)

进入bin目录，执行windows环境下的startup.cmd命令文件。

![1586151782862]({{site.url}}\imgs\springcloud\1586151782862.png)

nacos服务正常启动，可以看到启动默认端口为8848。

![1586151896016]({{site.url}}\imgs\springcloud\1586151896016.png)

打开 localhost:8848/nacos，默认账号 nacos,默认密码nacos，nacos服务运行完毕。

![1586151972713]({{site.url}}\imgs\springcloud\1586151972713.png)

#### 4.4.3 基于Nacos的服务注册

1. 首先要在父工程中引入spring-cloud-alibaba的依赖版本管理

   ```XML
   <dependencyManagement>
           <dependencies>
               <dependency>
                  <groupId>com.alibaba.cloud</groupId>
                  <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                  <version>2.1.0.RELEASE</version>
                  <type>pom</type>
                  <scope>import</scope>
               </dependency>
           </dependencies>
   <dependencyManagement>
   ```

2. 修改product服务的pom文件，引入spring-cloud-alibaba-nacos组件

   ```XML
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>
   ```

3. 修改配置文件，添加nacos服务地址配置

   ```YML
   spring:
     profiles: product1
     cloud:
       nacos:
         discovery:
           server-addr: 127.0.0.1:8848
   ```

4. 重新启动product服务，查看nacos控制台。

   ![1586152883923]({{site.url}}\imgs\springcloud\1586152883923.png)

#### 4.4.4 基于Nacos的服务发现





## 五、负载均衡

经过以上的学习，已经实现了服务的注册和服务发现。当启动某个服务的时候，可以通过HTTP的形式将信息注册到注册中心，并且可以通过SpringCloud提供的工具获取注册中心的服务列表。但是服务之间的调用还存在很多的问题，如何更加方便的调用微服务，多个微服务的提供者如何选择，如何负载均衡等。

### 5.1 负载均衡概述

#### 5.1.1 什么是负载均衡

在搭建网站时，如果单节点的web服务性能和可靠性都无法达到要求；或者是在使用外网服务时，经常担心被人攻破，一不小心就会有打开外网端口的情况，通常这个时候加入负载均衡就能有效解决服务问题。

负载均衡是一种基础的网络服务，其原理是通过运行在前面的负载均衡服务，按照指定的负载均衡算法，将流量分配到后端服务集群上，从而为系统提供并行扩展的能力。

负载均衡的应用场景包括流量包、转发规则以及后端服务，由于该服务有内外网个例、健康检查等功能，能够有效提供系统的安全性和可用性。

![image-20200220204458582]({{site.url}}\imgs\springcloud\image-20200220204458582.png)

#### 5.1.2 客户端负载均衡与服务端负载均衡

**服务端负载均衡**

先发送请求到负载均衡服务器或者软件，然后通过负载均衡算法，在多个服务器之间选择一个进行访问；即在服务器端再进行负载均衡算法分配。 Nginx就是服务端的负载均衡。

**客户端负载均衡**

客户端会有一个服务器地址列表，在发送请求前通过负载均衡算法选择一个服务器，然后进行访问，这是客户端、负载均衡；即在客户端就进行负载均衡算法分配。

### 5.2 基于Ribbon的负载均衡

#### 5.2.1 什么是Ribbon

是Netﬂixfa 发布的一个负载均衡器，有助于控制HTTP 和TCP客户端行为。在SpringCloud 中，Eureka一般配合Ribbon进行使用，Ribbon提供了客户端负载均衡的功能，Ribbon利用从Eureka中读取到的服务信息，在调用服务节点提供的服务时，会合理的进行负载。
在SpringCloud中可以将注册中心和Ribbon配合使用，Ribbon自动的从注册中心中获取服务提供者的列表信息，并基于内置的负载均衡算法，请求服务。

#### 5.2.2 Ribbon的主要作用

**（1）服务调用**
基于Ribbon实现服务调用，是通过拉取到的所有服务列表组成（服务名-请求路径的）映射关系。借助RestTemplate 最终进行调用。
**（2）负载均衡**
当有多个服务提供者时，Ribbon可以根据负载均衡的算法自动的选择需要调用的服务地址

#### 5.2.3 基于Ribbon进行远程调用

##### 5.2.3.1 注册多个服务

1. 修改product-server配置文件，利用profile配置多个服务。

   ```YML
   spring:
     application:
       name: product-server
   ---
   spring:
     profiles: product1
   server:
     port: 8001
   eureka:
     client:
       service-url:
         #设置连接的eureka服务地址
         defaultZone: http://eureka1:9000/eureka,http://eureka2:9001/eureka
     instance:
       #设置链结后的id显示信息为 ip:服务名称:端口号
       instance-id: ${spring.cloud.client.ip-address}:${spring.application.name}:${server.port}
       lease-expiration-duration-in-seconds: 10 #eureka client发送心跳给server端后，续 约到期时间（默认90秒）
       lease-renewal-interval-in-seconds: 5 #发送心跳续约间隔
       #设置显示 实例id的ip信息
       prefer-ip-address: true
   ---
   spring:
     profiles: product2
   server:
     port: 8003
   eureka:
     client:
       service-url:
         #设置连接的eureka服务地址
         defaultZone: http://eureka1:9000/eureka,http://eureka2:9001/eureka
     instance:
       #设置链结后的id显示信息为 ip:服务名称:端口号
       instance-id: ${spring.cloud.client.ip-address}:${spring.application.name}:${server.port}
       lease-expiration-duration-in-seconds: 10 #eureka client发送心跳给server端后，续 约到期时间（默认90秒）
       lease-renewal-interval-in-seconds: 5 #发送心跳续约间隔
       #设置显示 实例id的ip信息
       prefer-ip-address: true
   
   ```

2. 为了对不同服务的调用结果作区分，因而对查询商品的返回值进行修改。

   ```java
   @RestController
   @RequestMapping("product")
   public class ProductController {
   
       @Value("${server.port}")
       private String port;
       @Value("${spring.cloud.client.ip-address}")
       private String ip;
   
       @GetMapping("/{id}")
       public Product getPrduct(@PathVariable Integer id){
           Product product = null;
           if(id == 1){
               product = new Product();
               product.setName("调用ip"+ip+",端口-"+port);
               product.setPrice("50元");
               product.setId(1);
           }
           return product;
       }
   }
   ```

3. 启动两个商品服务，查看Eureka管理页面。

   ![image-20200220212110659]({{site.url}}\imgs\springcloud\image-20200220212110659.png)

   两个服务已经注册成功，分别为8001端口、8003端口。

##### 5.2.3.2 远程负载均衡调用服务

Ribbon是客户端负载均衡，因而使用的地方是服务调用方。在注册中心引入时，已经对Ribbon进行了依赖，因而不用再导入其他依赖。SpringCloud已经对注册中心使用Ribbon进行了封装，不管是Eureka还是Consul使用起来都是一样的。

1. 注入RestTemplate时添加额外注解，底层是通过AOP的方式，在RestTemplate调用服务时进行负载均衡的。

   ```java
   /**
    * 配置类
    *
    * @author Xue-Pan
    * @date 2020/2/18
    */
   @Configuration
   public class OrderConfig {
   	
       //添加LoadBalanced注解，即可使RestTemplate拥有负载均衡的效果
       @Bean
       @LoadBalanced
       public RestTemplate restTemplate(){
           return new RestTemplate();
       }
   }
   ```

2. 修改服务调用方式，不再直接使用服务的端口和ip,而是通过服务的应用名称来进行调用。

   ```java
   /**
    * 订单控制器
    *
    * @author Xue-Pan
    * @date 2020/2/18
    */
   @RestController
   @RequestMapping("order")
   public class OrderController {
       @Autowired
       private RestTemplate restTemplate;
   	//直接使用 http://+application.name+调用路径来调用
       @GetMapping("/{id:\\d+}")
       public Object getOrder(@PathVariable Integer id) {
           return restTemplate.getForObject("http://product-server/product/"+id, Product.class);
       }
   }
   ```

3. 启动order-client服务，并进行多次调用查看结果。

![image-20200220213916527]({{site.url}}\imgs\springcloud\image-20200220213916527.png)

![image-20200220213947742]({{site.url}}\imgs\springcloud\image-20200220213947742.png)

​		两个调用结果交替出现，使用的是轮询策略。

##### 5.2.3.3 Ribbon的负载策略

Ribbon内置了多种负载均衡策略，内部负责复杂均衡的顶级接口为
`com.netflix.loadbalancer.IRule` ，实现方式如下：

![image-20200220214250232]({{site.url}}\imgs\springcloud\image-20200220214250232.png)

- com.netflix.loadbalancer.RoundRobinRule：以轮询的方式进行负载均衡（默认）。 
- com.netflix.loadbalancer.RandomRule：随机策略
- com.netflix.loadbalancer.RetryRule：重试策略。
- com.netflix.loadbalancer.WeightedResponseTimeRule：权重策略。根据平均响应时间计算所有服务的权重，响应时间越快服务权重越大被选中的概率越高。刚启动时如果统计信息不足，则使用RoundRobinRule策略，等统计信息足够，会切换到WeightedResponseTimeRule
- com.netflix.loadbalancer.BestAvailableRule：最佳策略。遍历所有的服务实例，过滤掉故障实例，并返回请求数最小的实例返回。
- com.netflix.loadbalancer.AvailabilityFilteringRule：可用过滤策略。过滤掉故障和请求数超过阈值的服务实例，再从剩下的实例中轮询调用。

在服务消费者的application.yml配置文件中修改负载均衡策略

```YML
# 自定义负载均衡策略为随机
product-server:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

#### 5.2.4 Ribbon负载均衡源码解析

##### 5.2.4.1 Ribbon中的关键组件

![image-20200220220651944]({{site.url}}\imgs\springcloud\image-20200220220651944.png)

- ServerList：可以响应客户端的特定服务的服务器列表。
- ServerListFilter：可以动态获得的具有所需特征的候选服务器列表的过滤器。 ServerListUpdater：用于执行动态服务器列表更新。
- Rule：负载均衡策略，用于确定从服务器列表返回哪个服务器。
- Ping：客户端用于快速检查服务器当时是否处于活动状态。
- LoadBalancer：负载均衡器，负责负载均衡调度的管理。

##### 5.2.4.2 @LoadBalanced

开启Ribbon的负载均衡，是从给RestTemplate加上@LoadBalanced注解开始的。

```java
@Configuration
public class OrderConfig {
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

看看这个注解起了什么作用。

```java
/**
 * Annotation to mark a RestTemplate bean to be configured to use a LoadBalancerClient.
 	注解标记一个RestTemplate的实例，以使它被一个LoadBalancerClient配置
 * @author Spencer Gibb
 */
@Target({ ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Qualifier
public @interface LoadBalanced {
}
```

可见这个注解给这个RestTemplate实例进行了标记。

略。。。

### 5.3 基于Feign的远程调用

前面我们利用Ribbon在RestTemplate的基础上实现了远程调用的负载均衡，调用方式如下：

```java
@GetMapping("/{id:\\d+}")
    public Object getOrder(@PathVariable Integer id) {
        return restTemplate.getForObject("http://product-server/product/"+id, Product.class);
    }
```

虽然摆脱了ip+port的硬关联，但仍旧是基于url的调用方式，这与我们基于接口的服务调用习惯并不相符，于是我们需要引入另一个技术来解决这个问题。

#### 5.3.1 Feign简介

Feign是Netﬂix开发的声明式，模板化的HTTP客户端，其灵感来自Retroﬁt,JAXRS-2.0以及WebSocket，它可以使我们利用接口来完成API调用。

- Feign可帮助我们更加便捷，优雅的调用HTTP API。
- 在SpringCloud中，使用Feign非常简单——创建一个接口，并在接口上添加一些注解，代码就完成了。
- Feign支持多种注解，例如Feign自带的注解或者JAX-RS注解等。
- SpringCloud对Feign进行了增强，使Feign支持了SpringMVC注解，并整合了Ribbon和Eureka，从而让Feign的使用更加方便。

#### 5.3.2 Feign实现远程调用

1. 服务消费者引入依赖，order-client模块中引入Feign依赖

   ```XML
   <dependency>
   	<groupId>org.springframework.cloud</groupId>
   	<artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   ```

2. 启动类上增加开启Feign的注解。

   ```java
   /**
    * 订单启动类
    *
    * @author Xue-Pan
    * @date 2020/2/18
    */
   @SpringBootApplication
   @EnableFeignClients
   public class OrderApplication {
       public static void main(String[] args) {
           SpringApplication.run(OrderApplication.class, args);
       }
   }
   ```

3. 编写调用接口。@FeignClient(name=“product-server”)中name表示服务提供者的服务项目名称，各个API接口用RequestMapping与远程服务对应。

   ![image-20200221121504264]({{site.url}}\imgs\springcloud\image-20200221121504264.png)

   - 定义各参数绑定时，@PathVariable、@RequestParam、@RequestHeader等可以指定参数属性，在Feign中绑定参数必须通过value属性来指明具体的参数名，不然会抛出异常。

   - @FeignClient：注解通过name指定需要调用的微服务的名称，用于创建Ribbon的负载均衡器。所以Ribbon会把product-server 解析为注册中心的服务。

4. 改写远程服务调用写法，使用接口进行调用，删除无用的RestTemplate注入。

   ```java
   /**
    * 配置类
    *
    * @author Xue-Pan
    * @date 2020/2/18
    */
   @Configuration
   public class OrderConfig {
       //删除注入RestTemplate
   }
   
   /**
    * 订单控制器
    *
    * @author Xue-Pan
    * @date 2020/2/18
    */
   @RestController
   @RequestMapping("order")
   public class OrderController {
   	
       //注入商品服务类
       @Autowired
       private ProductService productService;
   
       @GetMapping("/{id:\\d+}")
       public Object getOrder(@PathVariable Integer id) {
           //使用商品服务类完成远程调用。
           return productService.getProductByid(id);
       }
   }
   ```

5. 测试调用，多次结果发现仍旧有Ribbon的负载均衡效果。

   ![image-20200221122206718]({{site.url}}\imgs\springcloud\image-20200221122206718.png)

   ![image-20200221122236200]({{site.url}}\imgs\springcloud\image-20200221122236200.png)

#### 5.3.3 Feign与Ribbon的联系

**Ribbon**是一个基于HTTP 和TCP 客户端的负载均衡的工具。它可以在客户端配置。RibbonServerList（服务端列表），使用HttpClient 或RestTemplate 模拟http请求，步骤相当繁琐。

**Feign** 是在Ribbon的基础上进行了一次改进，是一个使用起来更加方便的HTTP 客户端。采用接口的方式，只需要创建一个接口，然后在上面添加注解即可，将需要调用的其他服务的方法定义成抽象方法即可，不需要自己构建http请求。然后就像是调用自身工程的方法调用，而感觉不到是调用远程方法，使得编写客户端变得非常容易。

Feign中本身已经集成了Ribbon依赖和自动配置，因此我们不需要额外引入依赖，也不需要再注册RestTemplate 对象。另外，我们可以像上节课中讲的那样去配置Ribbon，可以**通过ribbon.xx 来进行全局配置。也可以通过服务名.ribbon.xx 来对指定服务配置**

#### 5.3.4 Feign高级

##### 5.3.4.1 Feign配置

从Spring Cloud Edgware开始，Feign支持使用属性自定义Feign。对于一个指定名称的Feign
Client（例如该Feign Client的名称为feignName ），Feign支持如下配置项：

```YML
feign:
 client:
  config:
    feignName: ##定义FeginClient的名称
      connectTimeout: 5000  # 相当于Request.Options
      readTimeout: 5000     # 相当于Request.Options
       # 配置Feign的日志级别，相当于代码配置方式中的Logger
      loggerLevel: full
       # Feign的错误解码器，相当于代码配置方式中的ErrorDecoder
      errorDecoder: com.example.SimpleErrorDecoder
       # 配置重试，相当于代码配置方式中的Retryer
      retryer: com.example.SimpleRetryer
       # 配置拦截器，相当于代码配置方式中的RequestInterceptor
      requestInterceptors:
        - com.example.FooRequestInterceptor
        - com.example.BarRequestInterceptor
      decode404: false
```

- **feignName**：FeginClient的名称

- **connectTimeout** ：建立链接的超时时长

- **readTimeout** ：读取超时时长

- **loggerLevel**: Fegin的日志级别

- **errorDecoder** ：Feign的错误解码器
- **retryer** ：配置重试
- **requestInterceptors** ：添加请求拦截器

- **decode404** ：配置熔断不处理404异常

##### 5.3.4.2 请求压缩

Spring Cloud Feign 支持对请求和响应进行GZIP压缩，以减少通信过程中的性能损耗。通过下面的参数即可开启请求与响应的压缩功能：

```YML
feign:
 compression:
  request:
    enabled: true # 开启请求压缩
  response:
    enabled: true # 开启响应压缩

```

同时，我们也可以对请求的数据类型，以及触发压缩的大小下限进行设置：

```YML
feign:
 compression:
  request:
    enabled: true # 开启请求压缩
    mime-types: text/html,application/xml,application/json # 设置压缩的数据类型
    min-request-size: 2048 # 设置触发压缩的大小下限
```

注：上面的数据类型、压缩大小下限均为默认值。

##### 5.3.4.3 日志级别

在开发或者运行阶段往往希望看到Feign请求过程的日志记录，默认情况下Feign的日志是没有开启的。要想用属性配置方式来达到日志效果，只需在application.yml 中添加如下内容即可：

```YML
feign:
  client:
    config:
      product-server:
        logger-level: FULL
logging:
  level:
    com.pansky.order.service.ProductService: debug
```

- `logging.level.xx : debug` : Feign日志只会对日志级别为debug的做出响应

- `feign.client.config.shop-service-product.loggerLevel` : 配置Feign的日志Feign有四种日志级别：
  - NONE【性能最佳，适用于生产】：不记录任何日志（默认值）
  - BASIC【适用于生产环境追踪问题】：仅记录请求方法、URL、响应状态代码以及执行时间
  - HEADERS：记录BASIC级别的基础上，记录请求和响应的header。
  - FULL【比较适用于开发及测试环境定位问题】：记录请求和响应的header、body和元数据。

![image-20200221135611477]({{site.url}}\imgs\springcloud\image-20200221135611477.png)

##### 5.3.4.4 Feign源码

通过上面的使用过程，@EnableFeignClients和@FeignClient两个注解就实现了Feign的功能，那我们从
@EnableFeignClients注解开始分析Feign的源码

略。。 基本就是引入自动配置，配合@FeignClient利用动态代理产生接口的实现类，然后完成接口调用。

## 六、服务熔断

### 6.1 微服务架构的高并发问题

通过注册中心已经实现了微服务的服务注册和服务发现，并且通过Ribbon实现了负载均衡，已经借助Feign可以优雅的进行微服务调用。那么我们编写的微服务的性能怎么样呢，是否存在问题呢？

#### 6.1.1 性能工具Jmeter

![image-20200221141357936]({{site.url}}\imgs\springcloud\image-20200221141357936.png)

Apache JMeter是Apache组织开发的基于Java的压力测试工具。用于对软件做压力测试，它最初被设计用于Web应用测试，但后来扩展到其他测试领域。它可以用于测试静态和动态资源，例如静态文件、Java 小服务程序、CGI 脚本、Java 对象、数据库、FTP 服务器，等等。JMeter 可以用于对服务器、网络或对象模拟巨大的负载，来自不同压力类别下测试它们的强度和分析整体性能。另外JMeter能够对应用程序做功能/回归测试，通过创建带有断言的脚本来验证你的程序返回了你期望的结果。为了最大限度的灵活性，JMeter允许使用正则表达式创建断言。

[官网](http://jmeter.apache.org/)

##### 6.1.1.1 下载安装

![image-20200221141655240]({{site.url}}\imgs\springcloud\image-20200221141655240.png)

如上图，点击Download Releases

![image-20200221144247159]({{site.url}}\imgs\springcloud\image-20200221144247159.png)

根据系统选择不同的包，下载解压即可。

##### 6.1.1.2 启动运行

参考[博客](https://blog.csdn.net/yaorongke/article/details/82799609)

#### 6.1.2 系统高并发可能出现的问题

##### 6.1.2.1 问题分析

在微服务架构中，我们将业务拆分成一个个的服务，服务与服务之间可以相互调用，由于网络原因或者自身的原因，并不能保证服务的100%可用，如果单个服务出现问题，调用这个服务就会出现网络延迟，此时若有大量的网络涌入，会形成任务累计，导致服务瘫痪。

在SpringBoot程序中，默认使用内置tomcat作为web服务器。单tomcat支持最大的并发请求是有限的，如果某一接口阻塞，待执行的任务积压越来越多，那么势必会影响其他接口的调用。

##### 6.1.2.2 简单的问题重现

1. 修改服务提供者，使查询商品延时2秒返回。

   ```java
   @GetMapping("/{id}")
       public Product getPrduct(@PathVariable Integer id){
           Product product = null;
           try {
               //在商品查询方法中睡眠2秒钟。
            Thread.sleep(2000);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
           if(id == 1){
               product = new Product();
               product.setName("调用ip"+ip+",端口-"+port);
               product.setPrice("50元");
               product.setId(1);
           }
           return product;
       }
   ```
   
2. 在服务消费者的controller中增加一个即时返回的方法。

   ```java
    @GetMapping("/xx")
   public Object getFastOrder(){
       return "调用成功";
   }
   ```

3. 为了方便观察，设置服务消费者tomcat的最大线程数为5。

   ```YML
   server:
     port: 8002
     tomcat:
       max-threads: 5
   feign:
     client:
       config:
         product-server:
           logger-level: FULL
           #设置超时时间，默认是2秒，会无法体现积压的请求时长。
           connectTimeout: 50000
           readTimeout: 50000
   ```

4. 使用Jmeter开启多线程循环请求查询商品,即调用 /order/1 接口。

5. 通过页面调用消费者的即时返回方法，发现耗时很久，原先 3ms,现在要12s左右。

原因： 大量请求调用延时的接口，线程无法及时处理，请求开始积压，导致正常的反应快的请求仍旧要排队，从而导致整个服务的接口崩溃。

解决办法：服务隔离，将不同api分割开来，分配各自的调用资源，当延时的接口资源用尽时不会影响到其他接口的调用。 服务隔离一般有两种实现方式 ：

	1. 线程池隔离，对不同的api分配各自的线程池，谁用完了谁排队。
 	2. 信号量隔离，即对api进行阈值设置，同时的请求超过阈值则直接报错，不让请求积压。

### 6.2 服务熔断知识

#### 6.2.1雪崩效应

在微服务架构中，一个请求需要调用多个服务是非常常见的。如客户端访问A服务，而A服务需要调用B服务，B服务需要调用C服务，由于网络原因或者自身的原因，如果B服务或者C服务不能及时响应，A服务将处于阻塞状态，直到B服务C服务响应。此时若有大量的请求涌入，容器的线程资源会被消耗完毕，导致服务瘫痪。服务与服务间的依赖性，故障会传播，造成连锁反应，会对整个微服务系统造成灾难性的严重后果，这就是服务故障的“雪崩”效应。

![image-20200222113617176]({{site.url}}\imgs\springcloud\image-20200222113617176.png)

雪崩是系统中的蝴蝶效应导致其发生的原因多种多样，有不合理的容量设计，或者是高并发下某一个方法响应变慢，亦或是某台机器的资源耗尽。从源头上我们无法完全杜绝雪崩源头的发生，但是雪崩的根本原因来源于服务之间的强依赖，所以我们可以提前评估，做好熔断，隔离，限流。

#### 6.2.2 服务隔离

顾名思义，它是指将系统按照一定的原则划分为若干个服务模块，各个模块之间相对独立，无强依赖。
当有故障发生时，能将问题和影响隔离在某个模块内部，而不扩散风险，不波及其它模块，不影响整体
的系统服务。

#### 6.2.3 熔断降级

熔断这一概念来源于电子工程中的断路器（Circuit Breaker）。在互联网系统中，当下游服务因访问压
力过大而响应变慢或失败，上游服务为了保护系统整体的可用性，可以暂时切断对下游服务的调用。这
种牺牲局部，保全整体的措施就叫做熔断。

![image-20200222113936380]({{site.url}}\imgs\springcloud\image-20200222113936380.png)

所谓降级，就是当某个服务熔断之后，服务器将不再被调用，此时客户端可以自己准备一个本地的fallback回调，返回一个缺省值，也可以理解为兜底。

#### 6.2.4 服务限流

限流可以认为服务降级的一种，限流就是限制系统的输入和输出流量已达到保护系统的目的。一般来说系统的吞吐量是可以被测算的，为了保证系统的稳固运行，一旦达到的需要限制的阈值，就需要限制流量并采取少量措施以完成限制流量的目的。比方：推迟解决，拒绝解决，或者者部分拒绝解决等等。

### 6.3 Hystrix

#### 6.3.1 Hystrix介绍

Hystrix是由Netﬂix开源的一个延迟和容错库，用于隔离访问远程系统、服务或者第三方库，防止级联失败，从而提升系统的可用性与容错性。Hystrix主要通过以下几点实现延迟和容错。

- 包裹请求：使用HystrixCommand包裹对依赖的调用逻辑，每个命令在独立线程中执行。这使用了设计模式中的“命令模式”。
- 跳闸机制：当某服务的错误率超过一定的阈值时，Hystrix可以自动或手动跳闸，停止请求该服务一段时间。
- 资源隔离：Hystrix为每个依赖都维护了一个小型的线程池（或者信号量）。如果该线程池已满，发往该依赖的请求就被立即拒绝，而不是排队等待，从而加速失败判定。
- 监控：Hystrix可以近乎实时地监控运行指标和配置的变化，例如成功、失败、超时、以及被拒绝的请求等。
- 回退机制：当请求失败、超时、被拒绝，或当断路器打开时，执行回退逻辑。回退逻辑由开发人员自行提供，例如返回一个缺省值。
- 自我修复：断路器打开一段时间后，会自动进入“半开”状态。

#### 6.3.2 基于RestTemplate的Hystrix

1. 复制order-client，重命名为 rest-order-client,处理好新模块的依赖。

2. 删除关于Feign的配置、依赖等、修改回RestTemplate远程调用。

   ```java
   @Configuration
   public class OrderConfig {
   
       @Bean
       @LoadBalanced
       public RestTemplate restTemplate(){
           return new RestTemplate();
       }
   }
   
   /**
    * 订单控制器
    *
    * @author Xue-Pan
    * @date 2020/2/18
    */
   @RestController
   @RequestMapping("order")
   public class OrderController {
       @Autowired
       private RestTemplate restTemplate;
   
   
       @GetMapping("/{id:\\d+}")
       //配置熔断的注解，通过fallbackMethod指定熔断时调用的方法。
       @HystrixCommand(fallbackMethod = "getOrderBack")
       public Object getOrder(@PathVariable Integer id) {
           return restTemplate.getForObject("http://product-server/product/"+id, Product.class);
       }
   	
       //熔断降级调用的方法
       public Object getOrderBack(Integer id){
           return "调用接口失败了";
       }
   }
   ```

3. 增加Hystrix依赖。

   ```XML
    <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
           </dependency>
           <dependency>
                  <groupId>org.springframework.cloud</groupId>
                  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
           </dependency>
       </dependencies>
   ```

4. 开启熔断。

   ```JAVA
   /**
    * 订单启动类
    *
    * @author Xue-Pan
    * @date 2020/2/18
    */
   @SpringBootApplication
   //开启熔断
   @EnableCircuitBreaker
   public class OrderApplication {
       public static void main(String[] args) {
           SpringApplication.run(OrderApplication.class, args);
       }
   }
   ```

5. 配置熔断降级业务逻辑。

@HystrixCommand 还可以标注在类上，用以给这个类中的所有服务指定统一的熔断降级方法。

#### 6.3.3 基于 Feign的Hystrix

SpringCloud Fegin默认已为Feign整合了hystrix，所以添加Feign依赖后就不用在添加hystrix，那么怎
么才能让Feign的熔断机制生效呢，只要按以下步骤开发：

1. 修改order-client的配置文件，开启hystrix。

   ```YML
   feign:
     client:
       config:
         product-server:
           logger-level: FULL
           #设置连接的超时时间，
           connectTimeout: 50000
           readTimeout: 50000
     hystrix:
     #开启hystrix熔断
       enabled: true
   #设置熔断的超时时间，默认是1s，获取不到则会执行降级方法。
   #因为前面给产品服务增加了2秒的睡眠，因而这里需要设置长一点。
   # 注意：上面feign设置的超时时间也应该大于2秒，否则会因为feign调用超时报错而执行降级方法。
   hystrix:
     command:
       default:
         execution:
           isolation:
             thread:
               timeoutInMilliseconds: 5000
   ```

2. 编写Feign接口（ProductService）调用的实现类。

   新建一个ProductService的实现类。

   ![image-20200222203350765]({{site.url}}\imgs\springcloud\image-20200222203350765.png)

   ```java
   @Component
   public class ProductServiceImpl implements ProductService {
       @Override
       public Product getProductByid(Integer id) {
           Product product = new Product();
           product.setName("调用了熔断降级方法");
           return product;
       }
   }
   ```

3. 修改PorductService接口，指定熔断降级方法。

   ```java
   //在注解中指定处理熔断降级的类。
   @FeignClient(name = "product-server",fallback = ProductServiceImpl.class)
   public interface ProductService {
   
       /**
        * 根据id查询商品
        * @param id 商品id
        * @return 商品对象
        */
       @GetMapping("/product/{id}")
       Product getProductByid(@PathVariable Integer id);
   }
   ```

4. 测试结果。

   远程调用出错时，会使用容错方法。这里强调一下，要调用成功，需要注意各个组件的调用超时时间。

#### 6.3.4 Hystrix dashboard

除了实现容错功能，Hystrix还提供了近乎实时的监控，HystrixCommand和HystrixObservableCommand在执行时，会生成执行结果和运行指标。比如每秒的请求数量，成功数量等。这些状态会暴露在Actuator提供的/health端点中。只需为项目添加spring-boot-actuator 依赖，重启项目，访问http://XXX/actuator/hystrix.stream ,即可看到实时的监控数据。

1. 导入依赖

   ```XML
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
   </dependency>
   <!-- 图表的依赖，提前引入 -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
   </dependency>
   ```

2. 修改配置文件，增加Actuator暴露接口。

   ```YML
   management:
     endpoints:
       web:
         exposure:
           include: '*'
   ```

3. 在启动类上增加引入熔断注解。

   ```java
   @SpringBootApplication
   @EnableFeignClients
   //增加熔断注解
   @EnableCircuitBreaker
   public class OrderApplication {
       public static void main(String[] args) {
           SpringApplication.run(OrderApplication.class, args);
       }
   }
   ```

4. 访问如下地址

   ![image-20200222215012785]({{site.url}}\imgs\springcloud\image-20200222215012785.png)



访问/hystrix.stream接口获取的都是已文字形式展示的信息。很难通过文字直观的展示系统的运行状态，所以Hystrix官方还提供了基于图形化的DashBoard（仪表板）监控平台。Hystrix仪表板可以显示每个断路器（被@HystrixCommand注解的方法）的状态。

在上面的基础上，在启动类上增加引入DashBoard注解

```java
@SpringBootApplication
@EnableFeignClients
@EnableCircuitBreaker
//DashBoard注解
@EnableHystrixDashboard
public class OrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }
}
```

重启项目，访问http://localhost:8002/hystrix

![image-20200222221102871]({{site.url}}\imgs\springcloud\image-20200222221102871.png)

再输入健康信息的地址，出现如下监控页面。

![image-20200222221006608]({{site.url}}\imgs\springcloud\image-20200222221006608.png)

#### 6.3.5 断路器聚合监控Turbine

在微服务架构体系中，每个服务都需要配置Hystrix DashBoard监控。如果每次只能查看单个实例的监控数据，就需要不断切换监控地址，这显然很不方便。要想看这个系统的Hystrix Dashboard数据就需要用到Hystrix Turbine。Turbine是一个聚合Hystrix 监控数据的工具，他可以将所有相关微服务的Hystrix 监控数据聚合到一起，方便使用。引入Turbine后，整个监控系统架构如下：
![image-20200222221257118]({{site.url}}\imgs\springcloud\image-20200222221257118.png)

1. 创建turbine-hystrix项目，并引入依赖

   ```XML
   <dependencies>
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
       </dependency>
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
       </dependency>
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
       </dependency>
   </dependencies>
   ```

   ![image-20200223115807778]({{site.url}}\imgs\springcloud\image-20200223115807778.png)

2. 配置多个微服务的hystrix监控

   ```YML
   server:
     port: 8080
   spring:
     application:
       name: turbine-hystrix
   eureka:
     client:
       service-url:
         defaultZone: http://eureka2:9001/eureka/,http://eureka1:9000/eureka/
   turbine:
   # 监控的服务名称
     app-config: order-client
     clusterNameExpression: "'default'"
   ```

3. 配置启动类。

   ```java
   **
    * turbine监控启动类
    *
    * @author Xue-Pan
    * @date 2020/2/23
    */
   @SpringBootApplication
   @EnableHystrixDashboard
   @EnableTurbine
   public class TurbineApplication {
       public static void main(String[] args) {
           SpringApplication.run(TurbineApplication.class, args);
       }
   }
   ```

4. 测试

   浏览器访问http://localhost:8080/hystrix 展示HystrixDashboard。并在url位置输入http://localhost:8080/turbine.stream，动态根据turbine.stream数据展示多个微服务的监控数据

#### 6.3.6 熔断器状态

熔断器有三个状态CLOSED 、OPEN 、HALF_OPEN 熔断器默认关闭状态，当触发熔断后状态变更为
OPEN ,在等待到指定的时间，Hystrix会放请求检测服务是否开启，这期间熔断器会变为HALF_OPEN 半
开启状态，熔断探测服务可用则继续变更为CLOSED 关闭熔断器。

![image-20200223120414249]({{site.url}}\imgs\springcloud\image-20200223120414249.png)



- **Closed**：关闭状态（断路器关闭），所有请求都正常访问。代理类维护了最近调用失败的次数，如果某次调用失败，则使失败次数加1。如果最近失败次数超过了在给定时间内允许失败的阈值，则代理类切换到断开(Open)状态。此时代理开启了一个超时时钟，当该时钟超过了该时间，则切换到半断开（Half-Open）状态。该超时时间的设定是给了系统一次机会来修正导致调用失败的错误。
- **Open**：打开状态（断路器打开），所有请求都会被降级。Hystix会对请求情况计数，当一定时间内失败请求百分比达到阈值，则触发熔断，断路器会完全关闭。默认失败比例的阈值是50%，请求次数最少不低于20次。

- **Half Open**：半开状态，open状态不是永久的，打开后会进入休眠时间（默认是5S）。随后断路器会自动进入半开状态。此时会释放1次请求通过，若这个请求是健康的，则会关闭断路器，否则继续保持打开，再次进行5秒休眠计时。



熔断器的默认触发阈值是20次请求，不好触发。休眠时间时5秒，时间太短，不易观察，为了测试方
便，我们可以通过配置修改熔断策略：

```YML
hystrix:
  command:
    default:
      circuitBreaker:
      #出发熔断的最小请求数字默认20次  10秒内
        requestVolumeThreshold: 5
       	#熔断后10秒，去尝试请求一次。默认是5秒
        sleepWindowInMilliseconds: 10000
        #出发熔断的请求最小占比50%
        errorThresholdPercentage: 50
```

解读：

- requestVolumeThreshold：触发熔断的最小请求次数，默认20
- errorThresholdPercentage：触发熔断的失败请求最小占比，默认50%
- sleepWindowInMilliseconds：熔断多少秒后去尝试请求

当我们疯狂访问id为1的请求时（超过10次），就会触发熔断。断路器会端口，一切请求都会被降级处理。
此时你访问id为2的请求，会发现返回的也是失败，而且失败时间很短，只有20毫秒左右：

#### 6.3.7 熔断器的隔离策略

微服务使用Hystrix熔断器实现了服务的自动降级，让微服务具备自我保护的能力，提升了系统的稳定性，也较好的解决雪崩效应。其使用方式**目前支持两种策略**：

- **线程池隔离策略**：使用一个线程池来存储当前的请求，线程池对请求作处理，设置任务返回处理超时时间，堆积的请求堆积入线程池队列。这种方式需要为每个依赖的服务申请线程池，有一定的资源消耗，好处是可以应对突发流量（流量洪峰来临时，处理不完可将数据存储到线程池队里慢慢处理）
- **信号量隔离策略**：使用一个原子计数器（或信号量）来记录当前有多少个线程在运行，请求来先判断计数器的数值，若超过设置的最大线程个数则丢弃改类型的新请求，若不超过则执行计数操作请求来计数器+1，请求返回计数器-1。这种方式是严格的控制线程且立即返回模式，无法应对突发流量（流量洪峰来临时，处理的线程超过数量，其他的请求会直接返回，不继续去请求依赖的服务）

**线程池和型号量两种策略功能支持对比如下：**

![image-20200223121957395]({{site.url}}\imgs\springcloud\image-20200223121957395.png)

- `hystrix.command.default.execution.isolation.strategy` : 配置隔离策略
  - `ExecutionIsolationStrategy.SEMAPHORE` 信号量隔离
  - `ExecutionIsolationStrategy.THREAD` 线程池隔离

- `hystrix.command.default.execution.isolation.maxConcurrentRequests` : 最大信号量上
  限

### 6.4 Sentinel

18年底Netﬂix官方宣布Hystrix 已经足够稳定，不再积极开发Hystrix，该项目将处于维护模式。就目前来看Hystrix是比较稳定的，并且Hystrix只是停止开发新的版本，并不是完全停止维护，Bug什么的依然会维护的。因此短期内，Hystrix依然是继续使用的。但从长远来看，Hystrix总会达到它的生命周期，那么Spring Cloud生态中是否有替代产品呢？

Sentinel、Resilicence4J 。

#### 6.4.1 Sentinel介绍

Sentinel 是阿里巴巴开源的一款断路器实现，目前在Spring Cloud的孵化器项目Spring Cloud Alibaba中的一员Sentinel本身在阿里内部已经被大规模采用，非常稳定。因此可以作为一个较好的替代品。

![image-20200223122526941]({{site.url}}\imgs\springcloud\image-20200223122526941.png)

随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

Sentinel 具有以下特征:

- **丰富的应用场景**：Sentinel 承接了阿里巴巴近10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。
- **完备的实时监控**：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至500 台以下规模的集群的汇总运行情况。
- **广泛的开源生态**：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与SpringCloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入Sentinel。
- **完善的SPI 扩展点**：Sentinel 提供简单易用、完善的SPI 扩展接口。您可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等。

Sentinel的主要特征：

![image-20200224132327593]({{site.url}}\imgs\springcloud\image-20200224132327593.png)

#### 6.4.2 Sentinel与Hystrix的区别

|                | Sentinel                                                   | Hystrix               | resilience4j                     |
| -------------- | ---------------------------------------------------------- | --------------------- | -------------------------------- |
| 隔离策略       | 信号量隔离（并发线程数限流）                               | 线程池隔离/信号量隔离 | 信号量隔离                       |
| 熔断降级策略   | 基于响应时间、异常比率、异常数                             | 异常比率              | 基于异常比率、响应时间           |
| 实时统计实现   | 滑动窗口（LeapArray）                                      | 滑动窗口（RxJava）    | Ring Bit Buffer                  |
| 动态规则配置   | 支持多种数据源                                             | 支持多种数据源        | 有限支持                         |
| 扩展性         | 多个扩展点                                                 | 插件形式              | 接口形式                         |
| 基于注解的支持 | 支持                                                       | 支持                  | 支持                             |
| 限流           | 基于QPS、支持基于调用关系的限流                            | 有限的支持            | Rate Limiter                     |
| 流量整形       | 支持预热模式、匀速器模式、预热排队模式                     | 不支持                | 简单的Rate Limiter               |
| 系统自适应保护 | 支持                                                       | 不支持                | 不支持                           |
| 控制台         | 提供开箱即用的控制台，可配置规则、查看秒级监控、机器发现等 | 简单的监控查看        | 不提供控制台，可对接其它监控系统 |

#### 6.4.3 迁移方案

| Hystrix 功能          | 迁移方案                                                     |
| --------------------- | ------------------------------------------------------------ |
| 线程池隔离/信号量隔离 | Sentinel 不支持线程池隔离；信号量隔离对应Sentinel 中的线程数限流，详见[此处](https://github.com/alibaba/Sentinel/wiki/Guideline:-从-Hystrix-迁移到-Sentinel#信号量隔离) |
| 熔断器                | Sentinel 支持按平均响应时间、异常比率、异常数来进行熔断降级。从Hystrix 的异常比率熔断迁移的步骤详见[此处](https://github.com/alibaba/Sentinel/wiki/Guideline:-从-Hystrix-迁移到-Sentinel#熔断降级) |
| Command创建           | 直接使用Sentinel SphU API 定义资源即可，资源定义与规则配置分离，详见[此处](https://github.com/alibaba/Sentinel/wiki/Guideline:-从-Hystrix-迁移到-Sentinel#command-迁移) |
| 规则配置              | 在Sentinel 中可通过API 硬编码配置规则，也支持多种动态规则源  |
| 注解支持              | Sentinel 也提供注解支持，可以很方便地迁移，详见此处          |
| 开源框架支持          | Sentinel 提供Servlet、Dubbo、Spring Cloud、gRPC 的适配模块，开箱即用；若之前使用Spring Cloud Netﬂix，可迁移至[Spring Cloud Alibaba](https://github.com/alibaba/spring-cloud-alibaba) |

#### 6.4.4 名词解释

Sentinel 可以简单的分为Sentinel 核心库和Dashboard。核心库不依赖Dashboard，但是结合Dashboard 可以取得最好的效果。

使用Sentinel 来进行熔断保护，主要分为几个步骤:
1. 定义资源
2. 定义规则
3. 检验规则是否生效

**资源**：可以是任何东西，一个服务，服务里的方法，甚至是一段代码。

**规则**：Sentinel 支持以下几种规则：流量控制规则、熔断降级规则、系统保护规则、来源访问控制规则和热点参数规则。Sentinel 的所有规则都可以在内存态中动态地查询及修改，修改之后立即生效

先把可能需要保护的资源定义好，之后再配置规则。也可以理解为，只要有了资源，我们就可以在任何时候灵活地定义各种流量控制规则。在编码的时候，只需要考虑这个代码是否需要保护，如果需要保护，就将之定义为一个资源。

#### 6.4.5 Sentinel中的管理控制台

1. 从官网下载控制台jar包 https://github.com/alibaba/Sentinel/releases/download/1.6.3/sentinel-dashboard-1.6.3.jar

2. 启动控制

   ```SHELL
   java -Dserver.port=8080 -Dcsp.sentinel.dashboard.server=localhost:8080 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard.jar
   ```

   其中-Dserver.port=8080 用于指定Sentinel 控制台端口为8080 。
   从Sentinel 1.6.0 起，Sentinel 控制台引入基本的登录功能，默认用户名和密码都是sentinel 。可以参考鉴权模块文档配置用户名和密码。
   启动Sentinel 控制台需要JDK 版本为1.8 及以上版本。

#### 6.4.6  客户端接入

1. 引入jar包

   父工程springcloud-demo中引入,用来定义版本

   ```xml
   <dependencyManagement>
           <dependencies>
               <dependency>
                  <groupId>com.alibaba.cloud</groupId>
                  <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                  <version>2.1.0.RELEASE</version>
                  <type>pom</type>
                  <scope>import</scope>
               </dependency>
           </dependencies>
   <dependencyManagement>
   ```

   子工程 order-client引入sentinel依赖

   ```XML
   <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
   </dependency>
   ```

2. 配置启动参数。

   删除hystrix的配置及注解。增加sentinel的配置

   ```YML
   spring:
     cloud:
       sentinel:
         transport:
         #指定sentinel控制台的地址
           dashboard: localhost:8080
   ```

3. 登录控制台查看

   打开http://localhost:8080/，默认用户名sentinel密码sentinel。

   初次打开只能看到dashboard的监控。需要接入的客户端进行一次调用才会正常显示。`sentinel.eager=true` 

#### 6.4.7 通用资源保护

​	略

#### 6.4.8 基于rest的Sentinel熔断

​	略

#### 6.4.9 基于Feign的Sentinel熔断

1. 引入依赖。前面进行限流的时候已经引入了依赖

   ```XML
   <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
   </dependency>
   ```

2. 开启基于Sentinel的服务熔断。

   ```YML
   feign:
     sentinel:
       #开启基于sentinel的服务熔断
       enabled: true
   ```

3. 写熔断方法，测试。

基于Feign的Sentinel熔断与hystrix的区别就是

1. 依赖不同。

 	2. 开启的配置稍有区别。
 	3. 不用在配置类上添加 @EnableCircuitBreaker注解。

## 七、服务网关

### 7.1 服务网关概述

根据前面的技术，我们完成了服务之间的调用。客户端需要使用到几乎所有的服务，那么这些服务各自拥有各自的端口，甚至有些服务的地址也不相同，客户端如何来进行调用呢。

![image-20200228111455796]({{site.url}}\imgs\springcloud\image-20200228111455796.png)

如果让客户端直接与各个微服务通讯，可能会有很多问题

- 客户端会请求多个不同的服务，需要维护不同的请求地址，增加开发难度
- 在某些场景下存在跨域请求的问题
- 加大身份认证的难度，每个微服务需要独立认证

因此，我们需要一个微服务网关，介于客户端与服务器之间的中间层，所有的外部请求都会先经过微服务网关。客户端只需要与网关交互，只知道一个网关地址即可，这样简化了开发还有以下优点：

1. 易于监控
2. 易于认证
3. 减少了客户端与各个微服务之间的交互次数

![image-20200228111800628]({{site.url}}\imgs\springcloud\image-20200228111800628.png)

#### 7.1.1 服务网关概念

##### 7.1.1.1 什么是微服务网关

服务网关也叫API网关，API网关是一个服务器，是系统对外的唯一入口。API网关封装了系统内部架构，为每个客户端提供一个定制的API。API网关方式的核心要点是，所有的客户端和消费端都通过统一的网关接入微服务，在网关层处理所有的非业务功能。通常，网关也是提供REST/HTTP的访问API。服务端通过API-GW注册和管理服务。

##### 7.1.1.2 作用和应用场景

网关具有的职责，如身份验证、监控、负载均衡、缓存、请求分片与管理、静态响应处理。当然，最主要的职责还是与“外界联系”。

#### 7.1.2 常见的API网关

- **Kong**
  基于Nginx+Lua开发，性能高，稳定，有多个可用的插件(限流、鉴权等等)可以开箱即用。问题：只支持Http协议；二次开发，自由扩展困难；提供管理API，缺乏更易用的管控、配置方式。
- **Zuul**
  Netﬂix开源，功能丰富，使用JAVA开发，易于二次开发；需要运行在web容器中，如Tomcat。问题：缺乏管控，无法动态配置；依赖组件较多；处理Http请求依赖的是Web容器，性能不如Nginx；
- **Traeﬁk**
  Go语言开发；轻量易用；提供大多数的功能：服务路由，负载均衡等等；提供WebUI问题：二进制文件部署，二次开发难度大；UI更多的是监控，缺乏配置、管理能力；
- **Spring Cloud Gateway**
  SpringCloud提供的网关服务
- **Nginx+lua实现**
  使用Nginx的反向代理和负载均衡可实现对api服务器的负载均衡及高可用，问题：自注册的问题和网关本身的扩展性

#### 7.1.3 基于Nginx的网关实现

##### 7.1.3.1 Nginx介绍

![image-20200228113018835]({{site.url}}\imgs\springcloud\image-20200228113018835.png)

##### 7.1.3.2 正向/反向代理

1. **正向代理**

   ![image-20200228113114236]({{site.url}}\imgs\springcloud\image-20200228113114236.png)

   正向代理，"它代理的是客户端，代客户端发出请求"，是一个位于客户端和原始服务器(origin server)之间服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。客户端必须要进行一些特别的设置才能使用正向代理。

2. **反向代理**

   ![image-20200228113212705](F:\笔记\springcloud\imgs\image-20200228113212705.png)

   多个客户端给服务器发送的请求，Nginx服务器接收到之后，按照一定的规则分发给了后端的业务处理服务器进行处理了。此时~请求的来源也就是客户端是明确的，但是请求具体由哪台服务器处理的并不明确了，Nginx扮演的就是一个反向代理角色。客户端是无感知代理的存在的，反向代理对外都是透明的，访问者并不知道自己访问的是一个代理。因为客户端不需要任何配置就可以访问。反向代理，"它代理的是服务端，代服务端接收请求"，主要用于服务器集群分布式部署的情况下，反向代理隐藏了服务器的信息。

   如果只是单纯的需要一个最基础的具备转发功能的网关，那么使用Ngnix是一个不错的选择。

##### 7.1.3.3 准备工作

Nginx会用，后面有时间了再完善。

详见Nginx笔记。

### 7.2 Zuul

Nginx确实性能高，但很多微服务网关需要具备的功能它并不具备，比如权限效验，监控等功能。

#### 7.2.1 Zuul介绍

Zuul是Netﬂix开源的微服务网关，它可以和Eureka、Ribbon、Hystrix等组件配合使用，Zuul组件的核心是一系列的过滤器，这些过滤器可以完成以下功能：

- **动态路由**：动态将请求路由到不同后端集群
- **压力测试**：逐渐增加指向集群的流量，以了解性能
- **负载分配**：为每一种负载类型分配对应容量，并弃用超出限定值的请求
- **静态响应处理**：边缘位置进行响应，避免转发到内部集群
- **身份认证和安全**: 识别每一个资源的验证要求，并拒绝那些不符的请求。Spring Cloud对Zuul进行
  了整合和增强。
  Spring Cloud对Zuul进行了整合和增强

#### 7.2.2 搭建Zuul网关服务

Zuul是java语言开发的，所以使用本地idea就可以完成服务搭建。

1. 创建项目、导入依赖。

   在springclud-demo下创建子模块zuul-server,并导入依赖

   ```XML
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
   </dependency>
   ```

2. 编写启动类。

   ![image-20200228155130719]({{site.url}}\imgs\springcloud\image-20200228155130719.png)

   创建配置类ZuulApplication

   ```java
   /**
    * zuul服务启动类
    *
    * @author Xue-Pan
    * @date 2020/2/28
    */
   @SpringBootApplication
   //开启zuul的网关功能。
   @EnableZuulProxy
   public class ZuulApplication {
       public static void main(String[] args) {
           SpringApplication.run(ZuulApplication.class, args);
       }
   }
   ```

3. 编写配置。

   ```YML
   server:
     port: 80
   spring:
     application:
       name: zuul-server
   zuul:
   #路由配置
     routes:
     #路由名称，可自己定义
       product-server:
       #要路由的路径，即对什么路径进行路由。
         path: /product-server/**
         #路由到什么地址。
         url: http://127.0.0.1:8001
   ```

4. 访问测试。

   访问 http://localhost/product-server/product/1，成功打开product的服务结果。

##### 7.2.2.1 面向服务的路由

微服务一般是由几十、上百个服务组成，对于一个URL请求，最终会确认一个服务实例进行处理。如果对每个服务实例手动指定一个唯一访问地址，然后根据URL去手动实现请求匹配，这样做显然就不合理。

Zuul支持与Eureka整合开发，根据ServiceID自动的从注册中心中获取服务地址并转发请求，这样做的好处不仅可以通过单个端点来访问应用的所有服务，而且在添加或移除服务实例的时候不用修改Zuul的路由配置。

1. 增加Eureka的依赖。

   ```XML
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   ```

2. 开启Eureka注册中心（也可以不用显式标注）

   ```java
   /**
    * zuul服务启动类
    *
    * @author Xue-Pan
    * @date 2020/2/28
    */
   @SpringBootApplication
   @EnableZuulProxy
   //可加可不加
   @EnableEurekaClient
   public class ZuulApplication {
       public static void main(String[] args) {
           SpringApplication.run(ZuulApplication.class, args);
       }
   }
   ```

3. 修改配置文件，增加Eureka的配置，修改Zuul的路由配置。

   因为已经有了Eureka客户端，我们可以从Eureka获取服务的地址信息，因此映射时无需指定IP地址，而
   是通过服务名称来访问，而且Zuul已经集成了Ribbon的负载均衡功能。

   ```YML
   server:
     port: 80
   spring:
     application:
       name: zuul-server
   zuul:
     routes:
       product-server:
         path: /product-server/** #需要被路由的路径
         #url: http://127.0.0.1:8001 #路由的地址
         service-id: product-server #改由服务id来发现服务
   eureka:
     client:
       service-url:
       #配置eureka的服务中心地址。
         defaultZone: http://eureka1:9000/eureka,http://eureka2:9001/eureka
   ```

4. 访问测试。

   一样没毛病。

##### 7.2.2.2 简化路由配置

在刚才的配置中，我们的规则是这样的：

- zuul.routes.<route>.path=/xxx/** ：来指定映射路径。<route> 是自定义的路由名

- zuul.routes.<route>.serviceId=/product-service ：来指定服务名。

而大多数情况下，我们的<route> 路由名称往往和服务名会写成一样的。因此Zuul就提供了一种简化的
配置语法：zuul.routes.<serviceId>=<path>
上面的配置可以简化为一条：

```YML
zuul:
  routes:
    product-server: /product-server/**
```

##### 7.2.2.3 默认的路由规则

在使用Zuul的过程中，上面讲述的规则已经大大的简化了配置项。但是当服务较多时，配置也是比较繁琐的。因此Zuul就指定了默认的路由规则：

默认情况下，一切服务的映射路径就是服务名本身。
例如服务名为：service-product ，则默认的映射路径就是：/service-product/**

##### 7.2.2.4 加入Zuul后的架构图

![image-20200228164741354]({{site.url}}\imgs\springcloud\image-20200228164741354.png)

#### 7.2.3 Zuul的过滤器

通过之前的学习，我们得知Zuul它包含了两个核心功能：对请求的路由和过滤。其中路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础；而过滤器功能则负责对请求的处理过程进行干预，是实现请求校验、服务聚合等功能的基础。其实，路由功能在真正运行时，它的路由映射和请求转发同样也由几个不同的过滤器完成的。所以，过滤器可以说是Zuul实现API网关功能最为核心的部件，每一个进入Zuul的HTTP请求都会经过一系列的过滤器处理链得到请求响应并返回给客户端。

那么接下来，我们重点学习的就是Zuul的第二个核心功能：**过滤器**。

##### 7.2.3.1 ZuulFilter简介

Zuul 中的过滤器跟我们之前使用的javax.servlet.Filter 不一样，javax.servlet.Filter 只有一种类型，可以通过配置urlPatterns 来拦截对应的请求。而Zuul 中的过滤器总共有4 种类型，且每种类型都有对应的使用场景。
1. **PRE**：这种过滤器在请求被路由之前调用。我们可利用这种过滤器实现身份验证、在集群中选择请
求的微服务、记录调试信息等。
2. **ROUTING**：这种过滤器将请求路由到微服务。这种过滤器用于构建发送给微服务的请求，并使用
Apache HttpClient或Netﬁlx Ribbon请求微服务。
3. **POST**：这种过滤器在路由到微服务以后执行。这种过滤器可用来为响应添加标准的HTTP
Header、收集统计信息和指标、将响应从微服务发送给客户端等。
4. **ERROR**：在其他阶段发生错误时执行该过滤器。

Zuul提供了自定义过滤器的功能实现起来也十分简单，只需要编写一个类去实现zuul提供的接口

```java
public abstract ZuulFilter implements IZuulFilter{
  
    abstract public String filterType();
   
    abstract public int filterOrder();
   
    boolean shouldFilter();// 来自IZuulFilter
  
    Object run() throws ZuulException;// IZuulFilter
}
```

ZuulFilter是过滤器的顶级父类。在这里我们看一下其中定义的4个最重要的方法

- **shouldFilter** ：返回一个Boolean 值，判断该过滤器是否需要执行。返回true执行，返回false
  不执行。
- **run** ：过滤器的具体业务逻辑。
- **filterType** ：返回字符串，代表过滤器的类型。包含以下4种：
  - pre ：请求在被路由之前执行
  - routing ：在路由请求时调用
  - post ：在routing和errror过滤器之后调用
  - error ：处理请求时发生错误调用

- **filterOrder** ：通过返回的int值来定义过滤器的执行顺序，数字越小优先级越高。

##### 7.2.3.2 ZuulFilter声明周期

![image-20200228183915184]({{site.url}}\imgs\springcloud\image-20200228183915184.png)

- **正常流程**：

  - 请求到达首先会经过pre类型过滤器，而后到达routing类型，进行路由，请求就到达真正的
    服务提供者，执行请求，返回结果后，会到达post过滤器。而后返回响应。
  - 异常流程：
    整个过程中，pre或者routing过滤器出现异常，都会直接进入error过滤器，再error处理完毕后，会将请求交给POST过滤器，最后返回给用户。
  - 如果是error过滤器自己出现异常，最终也会进入POST过滤器，而后返回。
  - 如果是POST过滤器出现异常，会跳转到error过滤器，但是与pre和routing不同的时，请求不会再到达POST过滤器了。

- **不同过滤器的场景**：

  - 请求鉴权：一般放在pre类型，如果发现没有访问权限，直接就拦截了
  - 异常处理：一般会在error类型和post类型过滤器中结合来处理。
  - 服务调用时长统计：pre和post结合使用。

- **所有内置过滤器列表**：

  ![image-20200228184202530]({{site.url}}\imgs\springcloud\image-20200228184202530.png)

##### 7.2.3.3 自定义过滤器

```java
package com.pansky.zuul.filter;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.exception.ZuulException;
import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;
import org.springframework.http.HttpStatus;

import javax.servlet.http.HttpServletRequest;

/**
 * 自定义ZuulFilter
 *
 * @author Xue-Pan
 * @date 2020/2/28
 */
public class MyFilter extends ZuulFilter {
    /**
     * 返回filter类型，一种有4种
     *
     * @return
     */
    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    /**
     * filter的执行顺序
     *
     * @return
     */
    @Override
    public int filterOrder() {
        return 1;
    }

    /**
     * filter是否启用
     *
     * @return
     */
    @Override
    public boolean shouldFilter() {
        return true;
    }

    /**
     * filter执行的逻辑
     *
     * @return
     * @throws ZuulException
     */
    @Override
    public Object run() throws ZuulException {
        // 登录校验逻辑。
        // 1）获取Zuul提供的请求上下文对象
        RequestContext ctx = RequestContext.getCurrentContext();
        // 2) 从上下文中获取request对象
        HttpServletRequest req = ctx.getRequest();
        // 3) 从请求中获取token
        String token = req.getParameter("access-token");
        // 4) 判断
        if (token == null || "".equals(token.trim())) {
            // 没有token，登录校验失败，拦截
            ctx.setSendZuulResponse(false);
            // 返回401状态码。也可以考虑重定向到登录页。
            ctx.setResponseStatusCode(HttpStatus.UNAUTHORIZED.value());
        }
        // 校验通过，可以考虑把用户信息放入上下文，继续向后执行
        return null;
    }
}
```

RequestContext：用于在过滤器之间传递消息。它的数据保存在每个请求的ThreadLocal中。它用于存储请求路由到哪里、错误、HttpServletRequest、HttpServletResponse都存储在RequestContext中。RequestContext扩展了ConcurrentHashMap，所以，任何数据都可以存储在上下文中。

#### 7.2.4 源码解析

![image-20200228185246965]({{site.url}}\imgs\springcloud\image-20200228185246965.png)

在Zuul中，整个请求的过程是这样的，首先将请求给zuulservlet处理，zuulservlet中有一个zuulRunner对象，该对象中初始化了RequestContext：作为存储整个请求的一些数据，并被所有的zuulﬁlter共享。zuulRunner中还有FilterProcessor，FilterProcessor作为执行所有的zuulﬁlter的管理器。FilterProcessor从ﬁlterloader 中获取zuulﬁlter，而zuulﬁlter是被ﬁlterFileManager所加载，并支持groovy热加载，采用了轮询的方式热加载。有了这些ﬁlter之后，zuulservelet首先执行的Pre类型的过滤器，再执行route类型的过滤器，最后执行的是post 类型的过滤器，如果在执行这些过滤器有错误的时候则会执行error类型的过滤器。执行完这些过滤器，最终将请求的结果返回给客户端。

略

#### 7.2.5 Zuul存在的问题

在实际使用中我们会发现直接使用Zuul会存在诸多问题，包括：

- 性能问题
  - Zuul1x版本本质上就是一个同步Servlet，采用多线程阻塞模型进行请求转发。简单讲，每来一个请求，Servlet容器要为该请求分配一个线程专门负责处理这个请求，直到响应返回客户端这个线程才会被释放返回容器线程池。如果后台服务调用比较耗时，那么这个线程就会被阻塞，阻塞期间线程资源被占用，不能干其它事情。我们知道Servlet容器线程池的大小是有限制的，当前端请求量大，而后台慢服务比较多时，很容易耗尽容器线程池内的线程，造成容器无法接受新的请求。
- 不支持任何长连接，如websocket

#### 7.2.6 替换方案

Zuul2.x版本
SpringCloud Gateway

### 7.3 Gateway

Zuul 1.x 是一个基于阻塞IO 的API Gateway 以及Servlet；直到2018 年5 月，Zuul 2.x（基于Netty，也是非阻塞的，支持长连接）才发布，但Spring Cloud 暂时还没有整合计划。Spring Cloud Gateway 比Zuul 1.x 系列的性能和功能整体要好。

#### 7.3.1 Gateway简介

##### 7.3.1.1  简介

Spring Cloud Gateway 是Spring 官方基于Spring 5.0，Spring Boot 2.0 和Project Reactor 等技术开发的网关，旨在为微服务架构提供一种简单而有效的统一的API 路由管理方式，统一访问接口。SpringCloud Gateway 作为Spring Cloud 生态系中的网关，目标是替代Netﬂix ZUUL，其不仅提供统一的路由方式，并且基于Filter 链的方式提供了网关基本的功能，例如：安全，监控/埋点，和限流等。它是基于Nttey的响应式开发模式。

| 组件                 | RPS(request pre second) |
| -------------------- | ----------------------- |
| Spring cloud Gateway | Requests/sec: 32213.38  |
| Zuul 1X              | Requests/sec: 20800.13  |

上表为Spring Cloud Gateway与Zuul的性能对比，从结果可知，Spring Cloud Gateway的RPS是Zuul 1X的1.6倍。

##### 7.3.1.2 核心概念

![image-20200228190815969]({{site.url}}\imgs\springcloud\image-20200228190815969.png)

1. **路由（route）**路由是网关最基础的部分，路由信息由一个ID、一个目的URL、一组断言工厂和一组Filter组成。如果断言为真，则说明请求URL和配置的路由匹配。
2. **断言（predicates）**Java8中的断言函数，Spring Cloud Gateway中的断言函数输入类型是Spring5.0框架中的ServerWebExchange。Spring Cloud Gateway中的断言函数允许开发者去定义匹配来自Http Request中的任何信息，比如请求头和参数等。
3. **过滤器（ﬁlter）**一个标准的Spring webFilter，Spring Cloud Gateway中的Filter分为两种类型，分别是Gateway Filter和Global Filter。过滤器Filter可以对请求和响应进行处理。

#### 7.3.2  路由

##### 7.3.2.1 入门案例

1. 在springcloud-demo项目下创建子模块gateway。

   ![image-20200228195333657]({{site.url}}\imgs\springcloud\image-20200228195333657.png)

2. 导入依赖。

   ```XML
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-gateway</artifactId>
   </dependency>
   ```

   注意SpringCloud Gateway使用的web框架为webﬂux，和SpringMVC不兼容。引入的限流组件是hystrix。redis底层不再使用jedis，而是lettuce。

3. 编写启动类。

   ```java
   /**
    * gateway网关启动类
    *
    * @author Xue-Pan
    * @date 2020/2/28
    */
   @SpringBootApplication
   public class GatewayApplication {
       public static void main(String[] args) {
           SpringApplication.run(GatewayApplication.class, args);
       }
   }
   ```

4. 编写配置。

   ```YML
   server:
     port: 80
   spring:
     application:
       name: gateway-server
     cloud:
       gateway:
         routes:
           - id: product-server
             uri: http://127.0.0.1:8001
             #配置断言，路径为XXX的时候就路由，是将path直接添加到uri后面。
             predicates:
               - Path=/product/**
   ```

   - id：我们自定义的路由ID，保持唯一
   - uri：目标服务地址
   - predicates：路由条件，Predicate 接受一个输入参数，返回一个布尔值结果。该接口包含多种默认方法来将Predicate 组合成其他复杂的逻辑（比如：与，或，非）。
   - ﬁlters：过滤规则，暂时没用。

   上面这段配置的意思是，配置了一个id 为product-server的路由规则，当访问网关请求地址以product 开头时，会自动转发到地址：http://127.0.0.1:8001/ 。配置完成启动项目即可在浏览器访问进行测试，当我们访问地址http://localhost/product/1 时会去进行接口调用。

5. 启动并访问测试。

    http://localhost/product/1

##### 7.3.2.2 路由规则

Spring Cloud Gateway 的功能很强大，前面我们只是使用了predicates 进行了简单的条件匹配，其实Spring Cloud Gataway 帮我们内置了很多Predicates 功能。在Spring Cloud Gateway 中Spring 利用Predicate 的特性实现了各种路由匹配规则，有通过Header、请求参数等不同的条件来进行作为条件匹配到对应的路由。

![image-20200228200003945]({{site.url}}\imgs\springcloud\image-20200228200003945.png)

```YML
#路由断言之后匹配
spring:
cloud:
  gateway:
    routes:
    - id: after_route
      uri: https://xxxx.com
       #路由断言之前匹配
      predicates:
      - After=xxxxx
#路由断言之前匹配
spring:
cloud:
  gateway:
    routes:
    - id: before_route
      uri: https://xxxxxx.com
      predicates:
      - Before=xxxxxxx
#路由断言之间
spring:
cloud:
  gateway:
    routes:
    - id: between_route
      uri: https://xxxx.com
      predicates:
      - Between=xxxx,xxxx
#路由断言Cookie匹配,此predicate匹配给定名称(chocolate)和正则表达式(ch.p)
spring:
cloud:
  gateway:
    routes:
    - id: cookie_route
      uri: https://xxxx.com
      predicates:
      - Cookie=chocolate, ch.p
#路由断言Header匹配，header名称匹配X-Request-Id,且正则表达式匹配\d+
spring:
cloud:
  gateway:
    routes:
    - id: header_route
      uri: https://xxxx.com
      predicates:
      - Header=X-Request-Id, \d+
#路由断言匹配Host匹配，匹配下面Host主机列表,**代表可变参数
spring:
cloud:
  gateway:
    routes:
    - id: host_route
      uri: https://xxxx.com
      predicates:
      - Host=**.somehost.org,**.anotherhost.org
#路由断言Method匹配，匹配的是请求的HTTP方法
spring:
cloud:
  gateway:
    routes:
    - id: method_route
      uri: https://xxxx.com
      predicates:
      - Method=GET
#路由断言匹配，{segment}为可变参数
spring:
cloud:
  gateway:
    routes:
    - id: host_route
      uri: https://xxxx.com
      predicates:
      - Path=/foo/{segment},/bar/{segment}
#路由断言Query匹配，将请求的参数param(baz)进行匹配，也可以进行regexp正则表达式匹配(参数包含
foo,并且foo的值匹配ba.)
spring:
cloud:
  gateway:
    routes:
    - id: query_route
      uri: https://xxxx.com
      predicates:
      - Query=baz 或Query=foo,ba.
       
#路由断言RemoteAddr匹配，将匹配192.168.1.1~192.168.1.254之间的ip地址，其中24为子网掩码位
数即255.255.255.0  
spring:
cloud:
  gateway:
    routes:
    - id: remoteaddr_route
      uri: https://example.org
      predicates:
      - RemoteAddr=192.168.1.1/24
```

##### 7.3.2.3 动态路由

从入门案例中我们可以看到，仅仅的Gateway,配置中的uri是写死的调用地址，这里我们仍旧可以集成Eureka，来使用服务名称发现服务，完成动态路由的效果。

1. 引入Eureka的依赖。

   ```XML
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   ```

2. 修改配置文件。

   ```YML
   server:
     port: 80
   spring:
     application:
       name: gateway-server
     cloud:
       gateway:
         routes:
           - id: product-server
           #此处lb开头表示通过服务名称来获取地址
             uri: lb://product-server
             predicates:
               - Path=/product/**
   #配置eureka服务端地址            
   eureka:
     client:
       service-url:
         defaultZone: http://eureka1:9000/eureka,http://eureka2:9001/eureka
   ```

3. 启动eureka注册中心（可写可不写）

4. 重启网关服务完成测试。

##### 7.3.2.4 重写转发路径

在SpringCloud Gateway中，路由转发是直接将匹配的路由path直接拼接到映射路径（URI）之后，那么在微服务开发中往往没有那么便利。这里就可以通过RewritePath机制来进行路径重写。

**改造案例：**

1. 修改配置文件

   ```XML
   spring:
     cloud:
       gateway:
         routes:
           - id: product-server
             uri: lb://product-server
             predicates:
             #修改匹配路径
              # - Path=/product/**
              - Path=/product-server/**
   ```

   此时产生的路由路径为 http://127.0.0.1:8001/product-server/** ，地址不正确，404。

2. 添加转发路径RewritePath配置

   ```YML
   spring:
     cloud:
       gateway:
         routes:
           - id: product-server
             uri: lb://product-server
             predicates:
               - Path=/product-server/**
               #使用过滤器替换路由路径。
             filters:
               - RewritePath=/product-server/(?<segment>.*),/$\{segment}
   ```

   通过RewritePath配置重写转发的url，将/product-server/(?.*)，重写为{segment}，然后转发到订单
   微服务。比如在网页上请求http://localhost/product-server/product/1，此时会将请求转发到htt
   p://127.0.0.1:8001/product/1（值得注意的是在yml文档中\$ 要写成$\ ）

##### 7.3.2.5 通过服务注册中心直接获取路由配置

修改配置文件，增加如下配置

```YML
spring:
  cloud:
    gateway:
    #服务发现
      discovery:
        locator:
        #注册中心的名称都是大写，通过此配置把名称转化为小写
          lower-case-service-id: true
        #开启从注册中心发现路由配置。  
          enabled: true
```

此配置会从注册中心自动发现服务，并将服务的名称当做path来进行路由。而且也进行了重写转发。

即此时我们访问 http://127.0.0.1/order-client/order/1 ，就会被路由转发到 http://127.0.0.1/order/1 。

#### 7.3.3 过滤器

Spring Cloud Gateway除了具备请求路由功能之外，也支持对请求的过滤。通过Zuul网关类似，也是通过过滤器的形式来实现的。那么接下来我们一起来研究一下Gateway中的过滤器

##### 7.3.3.1 过滤器基础

过滤器的声明周期：

Spring Cloud Gateway 的Filter 的生命周期不像Zuul 的那么丰富，它只有两个：“pre” 和“post”。

- **PRE**：这种过滤器在请求被路由之前调用。我们可利用这种过滤器实现身份验证、在集群中选择请求的微服务、记录调试信息等。
- **POST**：这种过滤器在路由到微服务以后执行。这种过滤器可用来为响应添加标准的HTTPHeader、收集统计信息和指标、将响应从微服务发送给客户端等。

![image-20200229141936104]({{site.url}}\imgs\springcloud\image-20200229141936104.png)

过滤器类型：

Spring Cloud Gateway 的Filter 从作用范围可分为另外两种GatewayFilter 与GlobalFilter。

- GatewayFilter：应用到单个路由或者一个分组的路由上。
- GlobalFilter：应用到所有的路由上。

##### 7.3.3.2 局部过滤器

局部过滤器（GatewayFilter），是针对单个路由的过滤器。可以对访问的URL过滤，进行切面处理。在Spring Cloud Gateway中通过GatewayFilter的形式内置了很多不同类型的局部过滤器。这里简单将Spring Cloud Gateway内置的所有过滤器工厂整理成了一张表格，虽然不是很详细，但能作为速览使用。如下：

| 过滤器工厂                  | 作用                                                         | 参数                                                         |
| --------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| AddRequestHeader            | 为原始请求添加Header                                         | Header的名称及值                                             |
| AddRequestParameter         | 为原始请求添加请求参数                                       | 参数名称及值                                                 |
| AddResponseHeader           | 为原始响应添加Header                                         | Header的名称及值                                             |
| DedupeResponseHeader        | 剔除响应头中重复的值                                         | 需要去重的Header名称及去重策略                               |
| Hystrix                     | 为路由引入Hystrix的断路器保护                                | HystrixCommand的名称                                         |
| FallbackHeaders             | 为fallbackUri的请求头中添加具体的异常信息                    | Header的名称                                                 |
| PreﬁxPath                   | 为原始请求路径添加前缀                                       | 前缀路径                                                     |
| PreserveHostHeader          | 为请求添加一个preserveHostHeader=true的属性，路由过滤器会检查该属性以决定是否要发送原始的Host | 无                                                           |
| RequestRateLimiter          | 用于对请求限流，限流算法为令牌桶                             | keyResolver、rateLimiter、statusCode、denyEmptyKey、emptyKeyStatus |
| RedirectTo                  | 将原始请求重定向到指定的URL                                  | http状态码及重定向的url                                      |
| RemoveHopByHopHeadersFilter | 为原始请求删除IETF组织规定的一系列Header                     | 过配置指定仅删除哪些Header                                   |
| RemoveRequestHeader         | 为原始请求删除某个Header                                     | Header名称                                                   |
| RemoveResponseHeader        | 为原始响应删除某个Header                                     | Header名称                                                   |
| RewritePath                 | 重写原始的请求路径                                           | 原始路径正则表达式以及重写后路径的正则表达式                 |
| RewriteResponseHeader       | 重写原始响应中的某个Header                                   | Header名称，值的正则表达式，重写后的值SaveSession            |
| SaveSession                 | 在转发请求之前，强制执行`WebSession::save`操作               | 无                                                           |
| secureHeaders               | 为原始响应添加一系列起安全作用的响应头                       | 无，支持修改这些安全响应头的值                               |
| SetPath                     | 修改原始的请求路径                                           | 修改后的路径                                                 |
| SetResponseHeader           | 修改原始响应中某个Header的值                                 | Header名称，修改后的值                                       |
| SetStatus                   | 修改原始响应的状态码                                         | HTTP 状态码，可以是数字，也可以是字符串                      |
| StripPreﬁx                  | 用于截断原始请求的路径                                       | 使用数字表示要截断的路径的数量                               |
| Retry                       | 针对不同的响应进行重试                                       | retries、statuses、methods、series                           |
| RequestSize                 | 设置允许接收最大请求包的大小。如果请求包大小超过设置的值，则返回`413 Payload TooLarge` | 请求包大小，单位为字节，默认值为5M                           |
| ModifyRequestBody           | 在转发请求之前修改原始请求体内容                             | 修改后的请求体内容                                           |
| ModifyResponseBody          | 修改原始响应体的内容                                         | 修改后的响应体内容                                           |

每个过滤器工厂都对应一个实现类，并且这些类的名称必须以GatewayFilterFactory 结尾，这是Spring Cloud Gateway的一个约定，例如AddRequestHeader 对应的实现类为AddRequestHeaderGatewayFilterFactory 。对于这些过滤器的使用方式可以参考官方文档。

##### 7.3.3.3 全局过滤器

全局过滤器（GlobalFilter）作用于所有路由，Spring Cloud Gateway 定义了Global Filter接口，用户可以自定义实现自己的Global Filter。通过全局过滤器可以实现对权限的统一校验，安全性验证等功能，并且全局过滤器也是程序员使用比较多的过滤器。

Spring Cloud Gateway内部也是通过一系列的内置全局过滤器对整个路由转发进行处理如下：

![image-20200229160327242]({{site.url}}\imgs\springcloud\image-20200229160327242.png)

##### 7.3.3.4 统一鉴权

内置的过滤器已经可以完成大部分的功能，但是对于企业开发的一些业务功能处理，还是需要我们自己编写过滤器来实现的，那么可以通过自定义filter，去完成统一的权限校验。

###### 7.3.3.4.1 鉴权逻辑

开发中的鉴权逻辑：

- 当客户端第一次请求服务时，服务端对用户进行信息认证（登录）
- 认证通过，将用户信息进行加密形成token，返回给客户端，作为登录凭证
- 以后每次请求，客户端都携带认证的token
- 服务端对token进行解密，判断是否有效。

![image-20200229163127448]({{site.url}}\imgs\springcloud\image-20200229163127448.png)

如上图，对于验证用户是否已经登录鉴权的过程可以在网关层统一检验。检验的标准就是请求中是否携
带token凭证以及token的正确性。

###### 7.3.3.4.2 代码实现

下面的我们自定义一个GlobalFilter，去校验所有请求的请求参数中是否包含“token”，如何不包含请求
参数“token”则不转发路由，否则执行正常的逻辑。

```java
@Component
public class AuthorizeFilter implements GlobalFilter, Ordered {
   @Override
public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
 //String url = exchange.getRequest().getURI().getPath();
 //忽略以下url请求
 //if(url.indexOf("/login") >= 0){
 // return chain.filter(exchange);
 // }
   String token = exchange.getRequest().getQueryParams().getFirst("token");
   if (StringUtils.isBlank(token)) {
       log.info( "token is empty ..." );
       exchange.getResponse().setStatusCode( HttpStatus.UNAUTHORIZED );
       return exchange.getResponse().setComplete();
  }
   return chain.filter(exchange);
}
@Override
public int getOrder() {
	return 0;
}
```
-  自定义全局过滤器需要实现GlobalFilter和Ordered接口。
- 在ﬁlter方法中完成过滤器的逻辑判断处理
- 在getOrder方法指定此过滤器的优先级，返回值越大级别越低
- ServerWebExchange 就相当于当前请求和响应的上下文，存放着重要的请求-响应属性、请求实例和响应实例等等。一个请求中的request，response都可以通过ServerWebExchange 获取
- 调用chain.filter 继续向下游执行

#### 7.3.4 网关限流

##### 7.3.4.1 常见限流算法

**计数器**

计数器限流算法是最简单的一种限流实现方式。其本质是通过维护一个单位时间内的计数器，每次请求计数器加1，当单位时间内计数器累加到大于设定的阈值，则之后的请求都被拒绝，直到单位时间已经过去，再将计数器重置为零。

![image-20200229163747520]({{site.url}}\imgs\springcloud\image-20200229163747520.png)

**漏桶算法**
漏桶算法可以很好地限制容量池的大小，从而防止流量暴增。漏桶可以看作是一个带有常量服务时间的单服务器队列，如果漏桶（包缓存）溢出，那么数据包会被丢弃。在网络中，漏桶算法可以控制端口的流量输出速率，平滑网络上的突发流量，实现流量整形，从而为网络提供一个稳定的流量。

![image-20200229163853349]({{site.url}}\imgs\springcloud\image-20200229163853349.png)

为了更好的控制流量，漏桶算法需要通过两个变量进行控制：一个是桶的大小，支持流量突发增多时可以存多少的水（burst），另一个是水桶漏洞的大小（rate）。
**令牌桶算法**

令牌桶算法是对漏桶算法的一种改进，桶算法能够限制请求调用的速率，而令牌桶算法能够在限制调用的平均速率的同时还允许一定程度的突发调用。在令牌桶算法中，存在一个桶，用来存放固定数量的令牌。算法中存在一种机制，以一定的速率往桶中放令牌。每次请求调用需要先获取令牌，只有拿到令牌，才有机会继续执行，否则选择选择等待可用的令牌、或者直接拒绝。放令牌这个动作是持续不断的进行，如果桶中令牌数达到上限，就丢弃令牌，所以就存在这种情况，桶中一直有大量的可用令牌，这时进来的请求就可以直接拿到令牌执行，比如设置qps为100，那么限流器初始化完成一秒后，桶中就已经有100个令牌了，这时服务还没完全启动好，等启动完成对外提供服务时，该限流器可以抵挡瞬时的100个请求。所以，只有桶中没有令牌时，请求才会进行等待，最后相当于以一定的速率执行。

![day3_Image_50]({{site.url}}\imgs\springcloud\day3_Image_50.bmp)

##### 7.3.4.2 基于Filter的限流

SpringCloudGateway官方就提供了基于令牌桶的限流支持。基于其内置的过滤器工厂RequestRateLimiterGatewayFilterFactory 实现。在过滤器工厂中是通过Redis和lua脚本结合的方式进行流量控制。