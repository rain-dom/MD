# Eureka

## 整体介绍

1. 背景：传统应用中，组件间的调用通过有规范约束的接口实现，但拆成微服务之后，每个微服务实例的网络地址可能动态变化，数量也会变化，此时需要一个中心化组件来进行服务的登记和管理。

2. 好处：不用关心有多少提供方

3. 服务注册与发现包括两部分，一个是服务器端，另一个是客户端。

4. Eureka：是Netflix的一个子模块，是一个RESTful风格的服务，用于服务发现和注册的基础组件，包括两个部分：服务器端、客户端

   **Server**是一个公共服务，为Client提供服务注册和发现的功能，维护注册到自身的Client的相关信息，同时提供接口给Client获取注册表中其他服务的信息

   **Client**将自己的服务信息通过一定的方式登记到Server上，这样其它服务可以发现自己，同时可以通过server获得其它服务的信息，同时内置了负载均衡器。

## 服务注册与发现

### server功能

1. 注册：将微服务信息注册到注册中心。
   发现：查询可用微服务列表及其网络地址。

2. 服务注册表：记录各个微服务信息，例如服务名称，ip，端口等。

   注册表提供 查询API（查询可用的微服务实例）和管理API（用于服务的注册和注销）。

3. 服务检查：定时检测已注册的服务，如发现某实例长时间无法访问，就从注册表中移除。

### client功能

1. 注册：每个微服务启动时，将自己的网络地址等信息注册到注册中心，注册中心会存储（内存中）这些信息。
2. 获取服务注册表：服务消费者从注册中心，查询服务提供者的网络地址，并使用该地址调用服务提供者，为了避免每次都查注册表信息，所以client会定时去server拉取注册表信息到缓存到client本地，这样即使eureka挂了，也能直接通过缓存中注册表的信息访问其它服务的端口。
3. 心跳：各个微服务与注册中心通过某种机制（心跳）通信，若注册中心长时间和服务间没有通信，就会注销该实例。
4. 调用：实际的服务调用，通过注册表，解析服务名和具体地址的对应关系，找到具体服务的地址，进行实际调用。

### Eureka 单节点搭建

1. pom.xml

   ```sh
   <dependency>
   	<groupId>org.springframework.cloud</groupId>
   	<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
   </dependency>
   
   有的教程中还引入spring-boot-starter-web，其实不用。因为上面的依赖已经包含了它。在pom中点此依赖进去，一共点4次spring-cloud-netflix-eureka-server，发现web的依赖。
   
   ```

2. application.yml

   ```sh
   eureka: 
     client:
       #是否将自己注册到Eureka Server,默认为true，由于当前就是server，故而设置成false，表明该服务不会向eureka注册自己的信息
       register-with-eureka: false
       #是否从eureka server获取注册信息，由于单节点，不需要同步其他节点数据，用false
       fetch-registry: false
       #设置服务注册中心的URL，用于client和server端交流
       service-url:                      
         defaultZone: http://localhost:7900/eureka/
   ```

3. application.properties

   

   ```properties
   #是否将自己注册到Eureka Server,默认为true，由于当前就是server，故而设置成false，表明该服务不会向eureka注册自己的信息
   eureka.client.register-with-eureka=false
   #是否从eureka server获取注册信息，由于单节点，不需要同步其他节点数据，用false
   eureka.client.fetch-registry=false
   #设置服务注册中心的URL，用于client和server端交流
   eureka.client.service-url.defaultZone=http://localhost:7900/eureka/
   ```

   

4. 代码

   ```sh
   启动类上添加此注解标识该服务为配置中心
   @EnableEurekaServer
   ```

5. PS：Eureka会暴露一些端点。端点用于Eureka Client注册自身，获取注册表，发送心跳。

6. 简单看一下eureka server控制台，实例信息区，运行环境信息区，Eureka Server自身信息区。



## Eureka高可用

高可用：可以通过运行多个Eureka server实例并相互注册的方式实现。Server节点之间会彼此增量地同步信息，从而确保节点中数据一致。

* 1.Eureka 间互相独立
  * 优点：只要有一个活着，就可以运行
  * 缺点：请求会变多
* 2.Eureka 间需要通信数据，降低数据一致性，但提高可用性。



从Eureka 拉取的数据会在本地缓存，即使Eureka 都挂到，client也可以根据缓存调用service

#### 搭建步骤

##### 1.准备

准备2个节点部署eureka，也可以单机部署

修改本机host文件，绑定一个主机名，单机部署时使用ip地址会有问题

##### 2.配置文件

etc/hosts文件添加

127.0.0.1 euk1.com
127.0.0.1 euk2.com

**节点 1:**

```properties
#是否将自己注册到其他Eureka Server,默认为true 需要
eureka.client.register-with-eureka=true
#是否从eureka server获取注册信息， 需要
eureka.client.fetch-registry=true
#设置服务注册中心的URL，用于client和server端交流
#此节点应向其他节点发起请求
eureka.client.serviceUrl.defaultZone=http://euk2.com:7902/eureka/
#主机名，必填
eureka.instance.hostname=euk1.com
#management.endpoint.shutdown.enabled=true
#web端口，服务是由这个端口处理rest请求的
server.port=7901

```

**节点 2:**

```properties
#是否将自己注册到其他Eureka Server,默认为true 需要
eureka.client.register-with-eureka=true
#是否从eureka server获取注册信息， 需要
eureka.client.fetch-registry=true
#设置服务注册中心的URL，用于client和server端交流
#此节点应向其他节点发起请求
eureka.client.serviceUrl.defaultZone=http://euk1.com:7902/eureka/
#主机名，必填
eureka.instance.hostname=ek2.com
#management.endpoint.shutdown.enabled=true
#web端口，服务是由这个端口处理rest请求的
server.port=7902

hostname是用来查找主机的，appname表示分组
```

**主配置文件**

~~~ properties
#分两次启动，分别启动euk1和euk2
spring.profiles.active=euk1
spring.application.name=EurekaServer
~~~

## Actuator监控应用

### 开启监控

server端是自带的

provider和consumer是不带的

   ```xml
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-actuator</artifactId>
 </dependency>
   ```

### 开启所有端点

在application.yml中加入如下配置信息，让所有端点可以被发现

*代表所有节点都加载

```properties
#开启所有端点
management.endpoints.web.exposure.include=*
```



### api端点功能

#### Health

会显示系统状态

{"status":"UP"}

#### ==shutdown== 

用来关闭节点

开启远程关闭功能

```properties
management.endpoint.shutdown.enabled=true
```



使用**Post方式请求端点，远程就可以调用关闭**

{

  "message": "Shutting down, bye..."

}

 

 autoconfig 

获取应用的自动化配置报告 
 beans 

获取应用上下文中创建的所有Bean 

 

#### configprops 

获取应用中配置的属性信息报告 

  

#### env 

获取应用所有可用的环境属性报告 

#### Mappings

 获取应用所有Spring Web的控制器映射关系报告

####  info 

获取应用自定义的信息 

#### metrics

返回应用的各类重要度量指标信息 

**Metrics**节点并没有返回全量信息，我们可以通过不同的**key**去加载我们想要的值

 metrics/jvm.memory.max



## Eureka原理

hostname需要再hosts文件中配置

defaultZone代表当前端口要注册到的注册中心地址

### Register

**服务注册**

想要参与服务注册发现的实例首先需要向Eureka服务器注册信息

注册在第一次心跳发生时提交

### Renew

**续租，心跳**
由client向server发起，节省sever的资源

Eureka客户需要每30秒发送一次心跳来续租
更新通知Eureka服务器实例仍然是活动的。如果服务器在**90秒**内没有看到更新，它将从其注册表中删除实例

### Fetch Registry

Eureka客户端从服务器获取注册表信息并将其缓存在本地。之后，客户端使用这些信息来查找其他服务。

通过获取上一个获取周期和当前获取周期之间的增量更新，可以定期(每30秒)更新此信息。

节点信息在服务器中保存的时间更长(大约3分钟)，因此获取节点信息时可能会再次返回相同的实例。Eureka客户端自动处理重复的信息。

在获得增量之后，Eureka客户端通过比较服务器返回的实例计数来与服务器协调信息，如果由于某种原因信息不匹配，则再次获取整个注册表信息。

### Cancel

Eureka客户端在关闭时向Eureka服务器发送取消请求。这将从服务器的实例注册表中删除实例。

### Communication mechanism

通讯机制

Http协议下的Rest请求

默认情况下Eureka使用Jersey和Jackson以及JSON完成节点间的通讯



## 实战使用

### 客户端配置选项

```properties
#续约发送间隔默认30秒，心跳间隔
eureka.instance.lease-renewal-interval-in-seconds=5
#表示eureka client间隔多久去拉取服务注册信息，默认为30秒，对于api-gateway，如果要迅速获取服务注册状态，可以缩小该值，比如5秒
eureka.client.registry-fetch-interval-seconds=5
# 续约到期时间（默认90秒）
eureka.instance.lease-expiration-duration-in-seconds=60
```

### 服务器端配置选项

```properties
#关闭自我保护模式
eureka.server.enable-self-preservation=false
#失效服务间隔
eureka.server.eviction-interval-timer-in-ms=3000
```

### Rest服务调用

/eureka/status 查看服务状态

#### 注册到eureka的服务信息查看

 get: {ip:port}/eureka/apps

#### 注册到eureka的具体的服务查看

 get: {ip:port}/eureka/apps/{appname}/{id}

#### 服务续约

 put：{ip:port}/eureka/apps/{appname}/{id}?lastDirtyTimestamp={}&status=up

#### 更改服务状态

 put：{ip:port}/eureka/apps/{appname}/{id}/status?lastDirtyTimestamp={}&value={UP/DOWN}
 对应eureka源码的：InstanceResource.statusUpdate

#### 删除状态更新

 delete：{ip:port}/eureka/apps/{appname}/{id}/status?lastDirtyTimestamp={}&value={UP/DOWN}

#### 删除服务

 delete: {ip:port}/eureka/apps/{appname}/{id}

### 元数据

Eureka的元数据有两种：标准元数据和自定义元数据。

标准元数据：主机名、IP地址、端口号、状态页和健康检查等信息

**自定义元数据**：可以使用eureka.instance.metadata-map配置，这些元数据可以在远程客户端中访问，但是一般不改变客户端行为，除非客户端知道该元数据的含义

根据设定的**meta**信息区分角色，从而做相应操作。

```properties
eureka.instance.metadata-map.dalao=mashibing
```



## 自我保护机制

### 机制

Eureka在CAP理论当中是属于AP ， 也就说当产生网络分区时，Eureka保证系统的可用性，但不保证系统里面数据的一致性

默认开启，服务器端容错的一种方式，即短时间心跳不到达仍不剔除服务列表里的节点

默认情况下，Eureka Server在一定时间内，没有接收到某个微服务心跳，会将某个微服务注销（90S）。但是当网络故障时，微服务与Server之间无法正常通信，上述行为就非常危险，因为微服务正常，不应该注销。

Eureka Server通过自我保护模式来解决整个问题，当Server在短时间内丢失过多客户端时，那么Server会进入自我保护模式，会保护注册表中的微服务不被注销掉。当网络故障恢复后，退出自我保护模式。

### 触发条件

**客户端每分钟续约数量小于客户端总数的85%时会触发保护机制**



### 关闭

```properties
eureka.server.enable-self-preservation=false
```



### 清理时间

默认60秒

```
eureka.server.eviction-interval-timer-in-ms=3000
```

## Eureka 健康检查

由于server和client通过心跳保持 服务状态，而只有状态为UP的服务才能被访问。看eureka界面中的status。

比如心跳一直正常，服务一直UP，但是此服务DB连不上了，无法正常提供服务。

此时，我们需要将 微服务的健康状态也同步到server。只需要启动eureka的健康检查就行。这样微服务就会将自己的健康状态同步到eureka。配置如下即可。

### 开启手动控制

在client端配置：将自己真正的健康状态传播到server。

==心跳没问题不代表服务没问题，在server的列表中status是UP的。服务中catch到问题通过actuator上报到server，关停服务==

```yaml
eureka:
  client:
    healthcheck:
      enabled: true
```

### Client端配置Actuator

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
```

## 安全配置

### 开启Eureka安全连接

==添加spring security组件==

```
spring.security.user.name=yiming
spring.security.user.password=123
```

![image-20200408185532993](G:/GIT/20 架构师三期 SpringCloud微服务架构/images/image-20200408185532993.png)

## 服务间调用

​		微服务中，很多服务系统都在独立的进程中运行，通过各个服务系统之间的协作来实现一个大项目的所有业务功能。服务系统间 使用多种跨进程的方式进行通信协作，而RESTful风格的网络请求是最为常见的交互方式之一。

spring cloud提供的方式：

1. RestTemplate
2. Feign

我个人习惯用RestTemplate，因为自由，方便调用别的第三方的http服务。feign也可以，更面向对象一些，更优雅一些，就是需要配置。



## ==调优==

#### server优化

* 自我保护

  * 1.阈值
    2.开关
  * 比如系数是80，也就是保护80%的节点，20个挂了，1个抖动，不会提出这个
  * 服务数量少时，服务挂了基本就是挂了，不开自我保护
    服务数量多时，服务挂了可能是网络波动，开自我保护

* 快速下线

  * Timer运行多个TimeTask时，只要其中之一没有捕获抛出的异常，其他任务便会自动终止运行，使用ScheduleExecutorService则没有这个问题
  * 减少服务上下线的延时

* 三级缓存（register，readWriteCacheMap，readOnlyCacheMap）

  * 新服务注册后会让readWriteCacheMap失效

  * 后两者的数据是30s同步一次的，为了高可用

  * ```yml
    #会直接从readWriteCacheMap拿数据，并发压力不高时设置
    use-read-only-response-cache: false
    #减少同步时间也能加快发现服务的速度
    response-cache-update-interval-ms: 1000
    ```

* 服务更新： 先停止，再发送下线请求，因为先下线可能会续约

~~~yml
server:	
	#自我保护
    enable-self-preservation: false
    #自我保护阈值
    renewal-percent-threshold: 0.85
    #快速下线（剔除服务时间间隔）
    eviction-interval-timer-in-ms: 1000
    #多级缓存
    use-read-only-response-cache: false
    #readWrite和readOnly同步时间间隔
    #减少同步时间也能加快发现服务的速度
    response-cache-update-interval-ms: 1000
~~~



#### client优化

注册，拉取，下线，3个定时任务

* heartbeatExecutor	心跳定时任务
  * cacheRefreshTask
  * heartbeatTask
  * statusChangeListener
    * 如果自身服务有变，重新注册



* 实际工作中，要把后面的url随机打乱
  * 因为默认是访问第一个，如果都配置一样的，会使得第一个压力过大
  * eureka会相互注册，所以会共享
    因此新来服务只注册第一个，就会被全部共享

```yml
server:
  port: 8080
eureka:
  #与server交互
  client:
    service-url:
      defaultZone: http://localhost:7900/eureka/
     
    #服务表拉取间隔时间
    registry-fetch-interval-seconds: 30
    #使与注册中心无关，禁用eureka-client
    enabled: false
  instance:
    #续约时间间隔
    lease-renewal-interval-in-seconds: 30

spring:
  application:
    name: api-passenger
logging:
  level: debug
```



### 服务测算

20个服务每个服务部署5个，eureka client：100个。
1分钟200。
心跳，向server发送我们活着，一天几十万次（对server来说）

集群并没有扩大承受能力，是为了高可用



### region

减少网络延迟



# Ribbon

## 简介

Spring Cloud Ribbon是一个基于HTTP和TCP的客户端负载均衡工具

Feign，它也是基于Ribbon实现的工具。

Ribbon是Netflix开发的客户端负载均衡器，为Ribbon配置**服务提供者地址列表**后，Ribbon就可以基于某种**负载均衡策略算法**，自动地帮助服务消费者去请求 提供者。Ribbon默认为我们提供了很多负载均衡算法，例如轮询、随机等。我们也可以实现自定义负载均衡算法。

## 负载均衡背景

### 两种策略

软件负载均衡分为：服务端（集中式），客户端。

**服务端**负载均衡：在客户端和服务端中间使用代理，nginx。
**客户端**负载均衡：根据自己的情况做负载。Ribbon就是。

客户端负载均衡和服务端负载均衡最大的区别在于 ***服务端地址列表的存储位置，以及负载算法在哪里***。

### 服务端负载均衡

在服务端负载均衡中，客户端节点只知道==单一服务代理==的地址，服务代理则知道所有服务端的地址。

### 客户端负载均衡

在客户端负载均衡中，==所有的客户端节点都有一份自己要访问的服务端地址列表==，这些列表统统都是从服务注册中心获取的；

---

上面几种负载均衡，硬件，软件（服务端nginx，客户端ribbon）。目的：将请求分发到其他功能相同的服务。



### 负载均衡算法

==默认==实现：

ZoneAvoidanceRule（区域权衡策略）：复合判断Server所在区域的性能和Server的可用性，轮询选择服务器。

其他规则：

BestAvailableRule（最低并发策略）：会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务。逐个找服务，如果断路器打开，则忽略。

RoundRobinRule（轮询策略）：以简单轮询选择一个服务器。按顺序循环选择一个server。

RandomRule（随机策略）：随机选择一个服务器。

AvailabilityFilteringRule（可用过滤策略）：会先过滤掉多次访问故障而处于断路器跳闸状态的服务和过滤并发的连接数量超过阀值得服务，然后对剩余的服务列表安装轮询策略进行访问。

WeightedResponseTimeRule（响应时间加权策略）：据平均响应时间计算所有的服务的权重，响应时间越快服务权重越大，容易被选中的概率就越高。刚启动时，如果统计信息不中，则使用RoundRobinRule(轮询)策略，等统计的信息足够了会自动的切换到WeightedResponseTimeRule。响应时间长，权重低，被选择的概率低。反之，同样道理。此策略综合了各种因素（网络，磁盘，IO等），这些因素直接影响响应时间。

RetryRule（重试策略）：先按照RoundRobinRule(轮询)的策略获取服务，如果获取的服务失败则在指定的时间会进行重试，进行获取可用的服务。如多次获取某个服务失败，就不会再次获取该服务。主要是在一个时间段内，如果选择一个服务不成功，就继续找可用的服务，直到超时。

## 使用

* 服务提供者只需要启动多个服务实例并注册到一个注册中心或是多个相关联的服务注册中心。
* 服务消费者直接通过调用被@LoadBalanced注解修饰过的RestTemplate来实现面向服务的接口调用。



### 切换负载均衡策略

#### 注解方式

```java
@Bean
	public IRule myRule(){
		//return new RoundRobinRule();
		//return new RandomRule();
		return new RetryRule(); 
```

#### 配置文件设置负载均衡策略

针对服务定ribbon策略：

```sh
provider.ribbon.NFLoadBalancerRuleClassName=com.netflix.loadbalancer.RandomRule

```

给所有服务定ribbon策略：

```sh
ribbon.NFLoadBalancerRuleClassName=com.netflix.loadbalancer.RandomRule
```

属性配置方式优先级高于Java代码。

### 超时

Feign默认支持Ribbon；Ribbon的重试机制和Feign的重试机制有冲突，所以源码中默认关闭Feign的重试机制,使用Ribbon的重试机制

==配在consumer端==

```properties
#连接超时时间(ms)
ribbon.ConnectTimeout=1000
#业务逻辑超时时间(ms)
ribbon.ReadTimeout=4000
```

### 重试

使用ribbon重试机制，请求失败后，每隔6秒会重新尝试

```properties
#同一台实例最大重试次数,不包括首次调用
ribbon.MaxAutoRetries=1
#重试负载均衡的其他实例最大重试次数,不包括首次调用
ribbon.MaxAutoRetriesNextServer=1
#是否所有操作都重试
ribbon.OkToRetryOnAllOperations=false
```

如果一个服务连续失败，ribbon会暂时停止对它的调用

# Hystrix熔断

## 概念

​	==在分布式系统下，微服务之间不可避免地会发生相互调用，但每个系统都无法百分之百保证自身运行不出问题。在服务调用中，很可能面临依赖服务失效的问题（网络延时，服务异常，负载过大无法及时响应）。因此需要一个组件，能提供强大的容错能力，为服务间调用提供保护和控制。==



我们的目的：***当我自身 依赖的服务不可用时，服务自身不会被拖垮。防止微服务级联异常***。

图。



本质：就是隔离坏的服务，不让坏服务拖垮其他服务（调用坏服务的服务）。

### 舱壁模式

舱壁模式（Bulkhead）隔离了每个工作负载或服务的关键资源，如连接池、内存和CPU，硬盘。每个工作单元都有独立的 连接池，内存，CPU。

使用舱壁避免了单个服务消耗掉所有资源，从而导致其他服务出现故障的场景。
这种模式主要是通过防止由一个服务引起的级联故障来增加系统的弹性。

### 雪崩效应

​		每个服务 发出一个HTTP请求都会 在 服务中 开启一个新线程。而下游服务挂了或者网络不可达，通常线程会阻塞住，直到Timeout。如果并发量多一点，这些阻塞的线程就会占用大量的资源，很有可能把自己本身这个微服务所在的机器资源耗尽，导致自己也挂掉。

​		如果服务提供者响应非常缓慢，那么服务消费者调用此提供者就会一直等待，直到提供者响应或超时。在高并发场景下，此种情况，如果不做任何处理，就会导致服务消费者的资源耗竭甚至整个系统的崩溃。一层一层的崩溃，导致所有的系统崩溃。

​		雪崩：由基础服务故障导致级联故障的现象。描述的是：提供者不可用 导致消费者不可用，并将不可用逐渐放大的过程。像滚雪球一样，不可用的服务越来越多。影响越来越恶劣。

服务不可用原因：

服务器宕机
网络故障
宕机
程序异常
负载过大，导致服务提供者响应慢
缓存击穿导致服务超负荷运行

==总之== ： 基础服务故障  导致 级联故障   就是  雪崩。

### 容错机制

1. 为网络请求设置超时。

   必须为网络请求设置超时。一般的调用一般在几十毫秒内响应。如果服务不可用，或者网络有问题，那么响应时间会变很长。长到几十秒。

   每一次调用，对应一个线程或进程，如果响应时间长，那么线程就长时间得不到释放，而线程对应着系统资源，包括CPU,内存，得不到释放的线程越多，资源被消耗的越多，最终导致系统崩溃。

   因此必须设置超时时间，让资源尽快释放。

2. 使用断路器模式。

   想一下家里的保险丝，跳闸。如果家里有短路或者大功率电器使用，超过电路负载时，就会跳闸，如果不跳闸，电路烧毁，波及到其他家庭，导致其他家庭也不可用。通过跳闸保护电路安全，当短路问题，或者大功率问题被解决，在合闸。

   自己家里电路，不影响整个小区每家每户的电路。

### 断路器

如果对某个微服务请求有大量超时（说明该服务不可用），再让新的请求访问该服务就没有意义，只会无谓的消耗资源。

**断路器==状态转换==的逻辑：**
		失败率达到阈值会打开断路器，一段时间后变为半开模式，允许一个服务发送请求。成功则关闭断路器，失败则打开断路器

> 关闭状态：正常情况下，断路器关闭，可以正常请求依赖的服务。
>
> 打开状态：当一段时间内，请求失败率达到一定阈值，断路器就会打开。服务请求不会去请求依赖的服务。调用方直接返回。不发生真正的调用。重置时间过后，进入半开模式。
>
> 半开状态：断路器打开一段时间后，会自动进入“半开模式”，此时，断路器允许一个服务请求访问依赖的服务。如果此请求成功(或者成功达到一定比例)，则关闭断路器，恢复正常访问。否则，则继续保持打开状态。

### 降级

为了在整体资源不够的时候，适当放弃部分服务，将主要的资源投放到核心服务中，待渡过难关之后，再重启已关闭的服务，保证了系统核心服务的稳定。当服务停掉后，自动进入fallback替换主方法。

用fallback方法代替主方法执行并返回结果，对失败的服务进行降级。当调用服务失败次数在一段时间内超过了断路器的阈值时，断路器将打开，不再进行真正的调用，而是==快速失败，直接执行fallback逻辑==。服务降级保护了服务调用者的逻辑。

>比如网约车中的计价服务，做了熔断，如果服务出现问题就返回默认10块



## Hystrix

### 概述

spring cloud 用的是 hystrix，是一个容错组件。

Hystrix实现了 超时机制和断路器模式。

### X单独使用

继承hystrixCommand

重写run

fallback（程序发生非HystrixBadRequestException异常，运行超时，熔断开关打开，线程池/信号量满了）

### 和restTemplate结合

#### 服务消费端

pom.xml

```sh
<!-- 引入hystrix依赖 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
		</dependency>
```

启动类

```sh
@EnableCircuitBreaker
```

Method
通过使用@HystrixCommand，将方法纳入到hystrix监控中。

```java
@HystrixCommand(fallbackMethod = "sendFail")

下面的service，功能只是：调用service-sms服务。
RestTemplateRequestServiceImpl中的smsSend
```



sendFail，此处需要注意：此方法的 请求参数和 返回参数 要和原方法一致。

```sh
	private ResponseResult sendFail(SmsSendRequest smsSendRequest) {
		
		//备用逻辑
		return ResponseResult.fail(-3, "熔断");
	}
```

### 配置

**示例**：

~~~java
@HystrixCommand(fallbackMethod = "sendFail",ignoreExceptions = {HystrixIgnoreException.class},
	commandProperties = {
			@HystrixProperty(name = "fallback.enabled",value = "false")			
	})
~~~

1. Execution：

   ~~~properties
   #该属性用来设置HystrixCommand.run()执行的隔离策略。默认为THREAD。
   execution.isolation.strategy：
   #该属性用来配置HystrixCommand执行的超时时间，单位为毫秒。
   execution.isolation.thread.timeoutInMilliseconds：
   ~~~

2. Fallback:

   ~~~properties
   #该属性用来设置服务降级策略是否启用，默认是true。如果设置为false，当请求失败或者拒绝发生时，将不会调用
   fallback.enabled：
   ~~~

3. Circuit Breaker(断路器)：

   ~~~properties
   #用来设置在滚动时间窗中，断路器熔断的最小请求数。例如，默认该值为20的时候，如果滚动时间窗（默认10秒）内仅收到19个请求，即使这19个请求都失败了，断路器也不会打开。
   circuitBreaker.requestVolumeThreshold：20
   
   #设置断路器打开后的休眠时间，到达时间变为半开状态
   circuitBreaker.sleepWindowInMilliseconds：
   
   #强制打开断路器，默认false
   circuitBreaker.forceClosed：true
   
   #触发断路器的错误比率阈值，默认50%
   circuitBreaker.errorThresholdPercentage：50
   ~~~

4. Request Context

   ~~~properties
   #用来配置是否开启请求缓存。
   requestCache.enabled：
   ~~~

### 捕获熔断的异常信息

1. restTemplate中：

在备用方法中 api-driver

```sh
	public ResponseResult sendFail(ShortMsgRequest shortMsgRequest,Throwable throwable) {
		log.info("异常信息："+throwable);
		//备用逻辑
		return ResponseResult.fail(-1, "熔断");
	}
```

加上一个Throwable，就Ok。

2. feign

   

**忽略异常**

> @HystrixCommand的范围大，中间可能出现很多异常，有些异常想忽略

1. 第一种方式：继承HystrixBadRequestException

```java
自定义异常，继承HystrixBadRequestException，当发生此异常时，不走备用方法。

public class BusinessException extends HystrixBadRequestException {
	
	private String message;
	
	public BusinessException(String message) {
		super(message);
		this.message = message;
	}
	
	/**
	 * 
	 */
	private static final long serialVersionUID = 1L;
	
}

在调用的地方前：
		// 下面是故意跑出异常代码
		try {
			int i = 1/0;
		} catch (Exception e) {
			// TODO: handle exception
			throw new BusinessException("熔断忽略的异常");
		}
```

2. 第二种方式：Hystrix属性配置。

   > 此时自定义的异常不需要继承hystrix的异常

   ~~~java
   //配置属性：
   @HystrixCommand(fallbackMethod = "sendFail",
      ignoreExceptions = {HystrixIgnoreException.class})
   
   //自定义异常：
   public class HystrixIgnoreException extends RuntimeException {
   	
   	private String message;
   	
   	public HystrixIgnoreException(String message) {
   		this.message = message;
   	}
   ~~~

   ### 断路器开关演示

   在项目中引入

   ```sh
   <dependency>
   			<groupId>org.springframework.boot</groupId>
   			<artifactId>spring-boot-starter-actuator</artifactId>
   		</dependency>
   ```

   访问健康地址：

   ```sh
   http://localhost:9002/actuator/health
   最开始：
   hystrix: {
   status: "UP"
   }
   
   
   HystrixCommandProperties   default_circuitBreakerRequestVolumeThreshold（在hyxtrix的properties中设置）
   10秒内，20次失败（20 requests in 10 seconds），则断路器打开。
   
   hystrix: {
   status: "CIRCUIT_OPEN",
   details: {
   openCircuitBreakers: [
   "SmsController::verifyCodeSend"
   ]
   }
   }
   ```

   ==相关的配置，主要是10秒20次，失败率超过 50%。==

### 可视化监控

在服务消费端 api-driver，配置actuator,jar

```sh
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

启动 eureka 7900，api-passenger 9002，service-sms 8002。

地址：http://localhost:8080/actuator/hystrix.stream



==上面的操作有点原始，刀耕火种。下面可视化。==

**项目**：hystrix-dashboard

> dashboard客户端，turbine也是通过这里来监控

pom

```sh
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
		</dependency>
```

启动类

```sh
@EnableHystrixDashboard
```

访问：http://localhost:6101/hystrix

输入：上面的地址：http://localhost:8080/actuator/hystrix.stream

面板说明：

github：https://github.com/Netflix-Skunkworks/hystrix-dashboard

解释：https://github.com/Netflix-Skunkworks/hystrix-dashboard/wiki



#### ==问题==

直接ping可以，用监控ping不上

### 集中可视化

上面的方法只能监控一个服务。实际生产中不方便。

**项目：study-hystrix-turbine**

pom

>要加入eureka客户端，因为监控所有服务

```pom
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-turbine</artifactId>
</dependency>
<!-- eureka客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
```

yml

```yml
eureka:


turbine:
  app-config: api-driver,api-passenger
  cluster-name-expression: "'default'"
```

启动类

```sh
@EnableTurbine
```

地址：http://localhost:6102/turbine.stream,

访问：http://localhost:6101/hystrix

填上上面的地址：http://localhost:6102/turbine.stream



# zuul网关

鉴权、限流

![网关架构](E:\deng\deng\MD\image\网关架构.png)

## 概念

Zuul是Netflix开源的微服务网关，核心是一系列过滤器。这些过滤器可以完成以下功能。

1. 是所有微服务入口，进行分发。

2. 身份认证与安全。识别合法的请求，拦截不合法的请求。

3. 监控。在入口处监控，更全面。

4. 动态路由。动态将请求分发到不同的后端集群。

5. 压力测试。可以逐渐增加对后端服务的流量，进行测试。

6. 负载均衡。也是用ribbon。

7. 限流（望京超市）。比如我每秒只要1000次，10001次就不让访问了。

zuul默认集成了：ribbon和hystrix。



网关也可以做两层，流量网关和业务网关

拒绝策略最好前置，不然做很多无用功

## 基本使用

### 初步使用

提醒自己：9100

启动 eureka 7900，api-driver 9002（服务提供者）, api-passenger 9001。

项目：online-taxi-zuul

pom

```pom
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>

		<!--zuul -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-zuul</artifactId>
		</dependency>
```

启动类

```sh
@EnableZuulProxy
该注解声明这是一个zuul代理，该代理使用Ribbon来定位注册到eureka server上的微服务，同时，整合了hystrix，实现了容错。
```

yml

```sh
普通配置，端口，应用名，eureka地址。即可

server:
  port: 9100

spring:
  application:
    name: online-taxi-zuul

#注册中心
eureka: 
  client:
    #设置服务注册中心的URL
    service-url:                      
      defaultZone: http://root:root@eureka-7901:7901/eureka/
  instance: 
    hostname: localhost
    instance-id: online-taxi-zuul 
```

测试点：

启动eureka，api-driver, zuul

1. 访问：online-taxi-zuul中，测试网关api-driver。

   ```sh
   http://网关ip:网关端口/服务名/微服务路径
   
   浏览器访问即可：
   http://localhost:9100/service-sms/test/sms-test
   
   相当于访问：
   http://localhost:9002/test/hello
   ```

结论：网关会将服务名转换成具体服务的ip和端口，实际进行访问。

注意：此处的ip和端口是  网关的ip和端口。



ps：网关一般命名

```sh
https://域名/v1/sms/路径

看高德开放平台：https://lbs.amap.com/api/webservice/guide/api/geofence_service#t6
```



### 负载均衡

启动两个api-driver-9002,  api-driver-9003。

测试点：

轮询访问上面地址：http://localhost:9100/api-driver/test/hello，会看到返回结果中，端口一直轮询在变。说明负载均衡生效了，默认是轮询。



改负载均衡

```sh
api-driver: 
  ribbon: 
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule

```

测试点：

轮询访问上面地址：http://localhost:9100/api-driver/test/hello，会看到返回结果中，端口一直随机在变。说明负载均衡生效了。

### 路由端点

zuul yml中

```sh
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    health:
      ##默认是never
      show-details: ALWAYS
      enabled: true
      
    #打开路由
    routes:
      enabled: true
```

访问：http://localhost:9100/actuator/routes

结果：

```sh
{
/gate-way/sms/**: "service-sms",
/api-driver/**: "api-driver"
}
```

结果中是，eureka中有的实例的网关，和自己配置的映射。如果eureka中没有实例，则只有自己配置的映射。



作用：调试的时候，看网关请求的地址，以及 映射是否正确。网关请求有误时，可以通过此处排查错误。

### 过滤器端点



访问：http://localhost:9100/actuator/filters

可以看到 如下4中过滤器，后面讲，网关就是==一系列过滤器==。每个类型的过滤器都罗列出来了。

有我们自己定义的，也有默认的。

```sh
error: [],
post: [],
pre: [],
route: []
```

### 配置指定微服务的访问路径

1. 通过服务名配置（虚拟主机名）

```sh
zuul:
  routes:
    api-driver: /zuul定制路径/test/hello
```

配置前先访问，然后做对比。

这样访问

http://localhost:9100/zuul-api-driver/test/hello



2. 自定义命名配置

```sh
zuul:
  routes:
    custom-zuul-name: #此处名字随便取
      path: /zuul-custom-name/**
      url: http://localhost:9002/
```

访问前 看结果，做对比。

访问：http://localhost:9100/zuul-custom-name/test/hello

这样 ribbon和hystrix 就都失效了。



3. 基于2，恢复ribbon+hystrix

```sh
zuul:
  routes:
    #此处名字随便取
    custom-zuul-name: 
      path: /zuul-custom-name/**
      service-id: no-eureka-api-driver
      
no-eureka-api-driver:
  ribbon: 
    listOfServers: localhost:9003,localhost:9002
ribbon: 
  eureka:
    enabled: false  
```

访问：http://localhost:9100/zuul-custom-name/test/hello

ribbon之前讲过类似这种配置。



4. 指定serviceId

```sh
zuul:
  routes:
    #此处名字随便取
    custom-zuul-name: 
      path: /zuul-custom-name/**
      service-id: api-driver
```

访问：http://localhost:9100/zuul-custom-name/test/hello

### 忽略微服务



### 前缀

接口一般命名：/api/v1/xxxx

```sh
zuul:
  prefix: /api
  # 是否移除前缀
  strip-prefix: true
```

访问时带上前缀，实际 请求会将前缀去掉。

比如访问：http://localhost:9100/api/zuul-api-driver/test/hello

实际：http://localhost:9002/test/hello



注意全局的移除，和自定义名字下面的移除。

即zuul下的移除，和custom-zuul-name2: 下面的移除。





### 敏感Header

测试点：

停止一个api-driver。访问：yapi：网关token，看返回。

初始请求。返回值中token为msb cookie



加上下面配置

敏感的header不会传播到下游去，也就是说此处的token不会传播的其它的微服务中去。

```sh
zuul:
  #一下配置，表示忽略下面的值向微服务传播，以下配置为空表示：所有请求头都透传到后面微服务。
  sensitive-headers: token
  
```

访问。网关token为null。



------

上面是网关的路由。



### 过滤器

#### 类型

**filterType：**pre，routing,post,error
**filterOrder:**执行顺序，越小优先级越高
**shouldFilter：**此过滤器是否执行，返回true就生效，可以写过滤器是否执行的判断条件。
**run：**具体执行逻辑。

```java
public class LimitFilter extends ZuulFilter {

    @Override
    public String filterType() {
        //过滤器类型
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        //越小优先级越高
        return -10;
    }

    @Override
    public boolean shouldFilter() {
        //这个是后面的过滤器要写的，根据limit值决定走不走
//        return (boolean) RequestContext.getCurrentContext().get("limit");

        return true;
    }

    // 2 qps(1秒  2个 请求 Query Per Second 每秒查询量)
    //
    private static final RateLimiter RATE_LIMITER = RateLimiter.create(50);

    @Override
    public Object run() throws ZuulException {
        RequestContext currentContext = RequestContext.getCurrentContext();

        if (RATE_LIMITER.tryAcquire()){
            System.out.println("通过");
            return null;
        }else {
            // 被流控的逻辑
            System.out.println("被限流了");
            currentContext.setSendZuulResponse(false);
            currentContext.setResponseStatusCode(HttpStatus.TOO_MANY_REQUESTS.value());
        }
        return null;
    }
}
```

### 接口容错

通过网关调用微服务失败时的类似fallback方法
getRoute`中返回支持回退的微服务name，*是支持所有
`fallbackResponse`，根据cause来走执行不同的`response`

~~~java
@Component
public class MsbFallback implements FallbackProvider{

	/**
	 * 表明为哪个微服务提供回退
	 * 服务Id ，若需要所有服务调用都支持回退，返回null 或者 * 即可
	 */
	@Override
	public String getRoute() {
		// TODO Auto-generated method stub
		return "*";
	}

	@Override
	public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
		//根据不同的cause给 response方法传入不同的H
		if (cause instanceof HystrixTimeoutException) {
            return response(HttpStatus.GATEWAY_TIMEOUT);
        } else {
            return response(HttpStatus.INTERNAL_SERVER_ERROR);
        }
		
		
	}
	
	private ClientHttpResponse response(final HttpStatus status) {
        return new ClientHttpResponse() {
            @Override
            public HttpStatus getStatusCode() throws IOException {
                //return status;
                return HttpStatus.BAD_REQUEST;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                //return status.value();
                return HttpStatus.BAD_REQUEST.value();
            }

            @Override
            public String getStatusText() throws IOException {
                //return status.getReasonPhrase();
                //return HttpStatus.BAD_REQUEST.name();
                return HttpStatus.BAD_REQUEST.getReasonPhrase();
            }

            @Override
            public void close() {
            }

            @Override
            public InputStream getBody() throws IOException {
                String msg = "{\"msg\":\"服务故障\"}";
            	return new ByteArrayInputStream(msg.getBytes());
            }

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                headers.setContentType(MediaType.APPLICATION_JSON);
                return headers;
            }
        };
    }

}


~~~



### 网关限流

使用**RateLimiter**令牌桶工具。`tryAcquire()`获取令牌状态，如果没拿到直接`setSendZuulResponse`为false，zuul直接返回false，不走后面的流程。

~~~java
@Component
public class LimitFilter extends ZuulFilter {

    //必须是PRE_TYPE
    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }


    @Override
    public int filterOrder() {
        //越小优先级越高
        return -10;
    }

    @Override
    public boolean shouldFilter() {
        //这个是后面的过滤器要写的，根据limit值决定走不走
//        return (boolean) RequestContext.getCurrentContext().get("limit");

        return true;
    }

    // 2 qps(1秒  2个 请求 Query Per Second 每秒查询量)
    //为限流设置参数
    private static final RateLimiter RATE_LIMITER = RateLimiter.create(50);

    @Override
    public Object run() throws ZuulException {
        //获取上下文
        RequestContext currentContext = RequestContext.getCurrentContext();

        //这个是非阻塞的，Acquire是阻塞的
        if (RATE_LIMITER.tryAcquire()){
            System.out.println("通过");
            return null;
        }else {
            // 被流控的逻辑
            System.out.println("被限流了");
            currentContext.setSendZuulResponse(false);
            currentContext.setResponseStatusCode(HttpStatus.TOO_MANY_REQUESTS.value());
        }
        return null;
    }
}
~~~

### 高可用

一般做法

前面架上nginx。