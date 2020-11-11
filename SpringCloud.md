## 基本启动

## 服务注册与发现

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



### Eureka高可用

高可用：可以通过运行多个Eureka server实例并相互注册的方式实现。Server节点之间会彼此增量地同步信息，从而确保节点中数据一致。

* 1.Eureka 间互相独立
  * 优点：只要有一个活着，就可以运行
  * 缺点：请求会变多
* 2.Eureka 间通信

降低数据一致性，从而提高可用性

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





