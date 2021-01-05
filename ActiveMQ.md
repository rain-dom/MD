## 应用场景

**异步处理：**不需要立即处理的事件放入队列，即使给用户反馈

**缓冲：**

**服务解耦：**后面的服务不影响前面的服务，同时前后可以使用不同的开发语言，只要他们遵守API接口统一的标准

**扩展性：**分布式扩容容易。

**异构系统：**

## 基本概念

queue、topic

> queue只能被一个consumer消费
> topic只能及时接受，但可以被多个消费

### Queue

队列存储，常用与点对点消息模型 

默认只能由唯一的一个消费者处理。一旦处理消息删除。 

### Topic

主题存储，用于订阅/发布消息模型

主题中的消息，会发送给所有的消费者同时处理。只有在消息可以重复处理的业务场景中可使用，消息不会持久化。

Queue/Topic都是 Destination 的子接口

### JMS的消息格式

通过sender发送的消息类型

#### JMS消息由以下三部分组成的：

- 消息头。

  每个消息头字段都有相应的getter和setter方法。

- 消息属性。

  如果需要除消息头字段以外的值，那么可以使用消息属性。

- 消息体。

  JMS定义的消息类型有TextMessage、MapMessage、BytesMessage、StreamMessage和ObjectMessage。

#### TextMessage

文本消息

#### MapMessage

k/v

#### BytesMessage

字节流

#### StreamMessage

java原始的数据流

#### ObjectMessage

序列化的java对象，继承序列化接口

### 基本使用

#### Sender

> 获取连接，注明目的地

```java
public class Sender {

	public static void main(String[] args) throws Exception {

		// 1.获取连接工厂
		ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory(
				"admin",
				"admin",
				"tcp://localhost:61616"
				);
		// 2.获取一个向ActiveMQ的连接
		Connection connection = connectionFactory.createConnection();
		// 3.获取session，先不用事务
		Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

//		以上三板斧固定

		// 4. 找目的地，获取destination，消费端，也会从这个目的地取消息
		Queue queue = session.createQueue("user");
		
		// 51.消息创建者
		
		MessageProducer producer = session.createProducer(queue);
	//	producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);


		// 5.2. 创建消息
		for (int i = 0; i < 1000; i++) {
			
			TextMessage textMessage = session.createTextMessage("hi: " + i);
			
			// 5.3 向目的地写入消息
			if(i % 4 == 0) {
				// 设置消息的优先级
				// 对producer 整体设置
			//	producer.setPriority(9);
			//	producer.send(textMessage,DeliveryMode.PERSISTENT,9,1000 * 100);
				textMessage.setJMSPriority(9);
				Thread.sleep(3000);

			}
			producer.send(textMessage);
//			Thread.sleep(3000);
		}
		
		// 6.关闭连接
		connection.close();
		
		System.out.println("System exit....");
		
	}
	
		
	
}
```

#### Receiver

~~~java
public class Receiver {

	public static void main(String[] args) throws Exception {
		// 1. 建立工厂对象，
		ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(
				ActiveMQConnectionFactory.DEFAULT_USER,
				ActiveMQConnectionFactory.DEFAULT_PASSWORD,
				"tcp://localhost:61616"
				);
		//2 从工厂里拿一个连接
		Connection connection = activeMQConnectionFactory.createConnection();
		connection.start();
		//3 从连接中获取Session(会话)
		Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
		// 从会话中获取目的地(Destination)消费者会从这个目的地取消息
		Queue queue = session.createQueue("user");
		
		
		//从会话中创建消息提供者
		
		MessageConsumer consumer = session.createConsumer(queue);
		//从会话中创建文本消息(也可以创建其它类型的消息体)
		


		while (true) {
			TextMessage receive = (TextMessage)consumer.receive();
			System.out.println("TextMessage:" + receive.getText());
			
		}
	}
}
~~~

## Active MQ的安全机制

### web控制台安全

jetty-reala.properties

```
# username: password [,rolename ...]
admin: admin, admin
user: user, user
yiming: 123, user
```

用户名：密码，**角色**

注意: 配置需重启ActiveMQ才会生效。

### 消息安全机制

修改 activemq.xml

在123行     </broker> 节点中添加

```xml
	<plugins>
      <simpleAuthenticationPlugin>
          <users>
              <authenticationUser username="admin" password="admin" groups="admins,publishers,consumers"/>
              <authenticationUser username="publisher" password="publisher"  groups="publishers,consumers"/>
              <authenticationUser username="consumer" password="consumer" groups="consumers"/>
              <authenticationUser username="guest" password="guest"  groups="guests"/>
          </users>
      </simpleAuthenticationPlugin>
 </plugins>

```



## JDBC存储 

> 数据先是存到内存中,消费也是先从内存取，异步去同步数据库
> 这样不如直接连mysql咯？
>
> 所以不会使用这种存储方式

使用JDBC持久化方式，数据库默认会创建3个表，每个表的作用如下： 
activemq_msgs：queue和topic的消息都存在这个表中 
activemq_acks：存储持久订阅的信息和最后一个持久订阅接收的消息ID 
activemq_lock：跟kahadb的lock文件类似，确保数据库在某一时刻只有一个broker在访问



ActiveMQ 将数据持久化到数据库中。 

不指定具体的数据库。 可以使用任意的数据库 中。 

本环节中使用 MySQL 数据库。 下述文件为 activemq.xml 配置文件部分内容。 

 首先定义一个 mysql-ds 的 MySQL 数据源，然后在 persistenceAdapter 节点中配置 jdbcPersistenceAdapter 并且引用刚才定义的数据源。

dataSource 指定持久化数据库的 bean，createTablesOnStartup 是否在启动的时候创建数 据表，默认值是 true，这样每次启动都会去创建数据表了，一般是第一次启动的时候设置为 true，之后改成 false。 



**Beans中添加**

> 用户名和密码，数据库位置改一改

```xml
<bean id="mysql-ds" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close"> 

<property name="driverClassName" value="com.mysql.jdbc.Driver"/> 
<property name="url" value="jdbc:mysql://localhost/activemq?relaxAutoCommit=true"/> 
<property name="username" value="root"/>
<property name="password" value="123456"/>
<property name="maxActive" value="200"/>
<property name="poolPreparedStatements" value="true"/> 

</bean>
```

**修改persistenceAdapter**

```XML
        <persistenceAdapter>
           <!-- <kahaDB directory="${activemq.data}/kahadb"/> -->

		<jdbcPersistenceAdapter dataSource="#mysql-ds" createTablesOnStartup="true" /> 


        </persistenceAdapter>
```

依赖jar包

commons-dbcp commons-pool mysql-connector-java



#### 表字段解释

**activemq_acks**：用于存储订阅关系。如果是持久化Topic，订阅者和服务器的订阅关系在这个表保存。
主要的数据库字段如下：

```
container：消息的destination 
sub_dest：如果是使用static集群，这个字段会有集群其他系统的信息 
client_id：每个订阅者都必须有一个唯一的客户端id用以区分 
sub_name：订阅者名称 
selector：选择器，可以选择只消费满足条件的消息。条件可以用自定义属性实现，可支持多属性and和or操作 
last_acked_id：记录消费过的消息的id。
```

2：**activemq_lock**：在集群环境中才有用，只有一个Broker可以获得消息，称为Master Broker，其他的只能作为备份等待Master Broker不可用，才可能成为下一个Master Broker。这个表用于记录哪个Broker是当前的Master Broker。

3：**activemq_msgs**：用于存储消息，Queue和Topic都存储在这个表中。
主要的数据库字段如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
id：自增的数据库主键 
container：消息的destination 
msgid_prod：消息发送者客户端的主键 
msg_seq：是发送消息的顺序，msgid_prod+msg_seq可以组成jms的messageid 
expiration：消息的过期时间，存储的是从1970-01-01到现在的毫秒数 
msg：消息本体的java序列化对象的二进制数据 
priority：优先级，从0-9，数值越大优先级越高 
xid:用于存储订阅关系。如果是持久化topic，订阅者和服务器的订阅关系在这个表保存。
```



### 配置项

~~~java
//开启事务
Session session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);
//设置不持久化
producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);

~~~



## 常用API

### 事务

消费消息时有session的概念，mq投递消息后会将消息状态改为待确认，session还没消失的话，消息就不会被重复投递，直到session结束还没有收到ack再讲消息改为未消费状态。


> 开启事务
> Session session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE)

```
session.commit();
session.rollback();
```

用来提交/回滚事务



手动确认消息，否则消息消费无效，仍可以被消费。
在acknowledge后没有commit期间，消息不会被重复消费，但是如果知道连接断开也没有commit，就可以再次被消费到了。

```java
message.acknowledge();
```

```java
for(int i=0;;i++) {
      
      TextMessage message = (TextMessage)consumer.receive();
      System.out.println("-----");
      System.out.println("message2:" + message.getText());

      // 事务中的一个操作
      message.acknowledge();
      // 必须commit,前面的操作才生效
      if(i % 4 == 0)
         session.commit();
      
   }
```

### 持久化

默认持久化是开启的

```
producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT)
```

### 优先级

可以打乱消费顺序
越高优先级越高

```
producer.setPriority
```

配置文件需要指定使用优先级的目的地

```
<policyEntry queue="queue1" prioritizedMessages="true" />
```



### **JMSReplyTo**

发送方可以接受到消息消费确认的地址

```java
TextMessage textMessage = session.createTextMessage(msg);
textMessage.setJMSReplyTo(new ActiveMQQueue("mama"));
```



## 消息类型

### object

需要序列化对象

#### 发送端

```java
		Cat cat = new Cat();
		ObjectMessage objectMessage = session.createObjectMessage(cat);
		producer.send(objectMessage);
```

#### 接受端

> 添加信任

```java
//1.1 添加信任的持久化类型
ArrayList<String> lists = new ArrayList<>();
lists.add(Cat.class.getPackage().getName());
connectionFactory.setTrustedPackages(lists);
```

~~~java
consumer.setMessageListener(new MyLisenter())；
~~~

~~~java
public class MyLisenter implements MessageListener {
    @Override
    public void onMessage(Message message) {
        if (message instanceof TextMessage){
            try {
                ((TextMessage) message).getText();
            } catch (JMSException e) {
                e.printStackTrace();
            }
        }else if (message instanceof ObjectMapper){
            try {
                Cat cat = (Cat)((ObjectMessage) message).getObject();
                System.out.println(cat);
            } catch (JMSException e) {
                e.printStackTrace();
            }
        }
    }
}
~~~

### MapMessage

#### 发送端

```java
		MapMessage mapMessage = session.createMapMessage();
      
		
		mapMessage.setString("name","lucy");
        mapMessage.setBoolean("yihun",false);
		mapMessage.setInt("age", 17);
		
		producer.send(mapMessage);
```

#### 接收端

```
		Message message = consumer.receive();
		MapMessage mes = (MapMessage) message;
		
		System.out.println(mes);

		System.out.println(mes.getString("name"));
```



## 消息发送原理

### 监听器

~~~java
consumer.setMessageListener(new MyLisenter())；
~~~

~~~java
public class MyLisenter implements MessageListener {
    @Override
    public void onMessage(Message message) {
        if (message instanceof TextMessage){
            try {
                ((TextMessage) message).getText();
            } catch (JMSException e) {
                e.printStackTrace();
            }
        }else if (message instanceof ObjectMapper){
            try {
                Cat cat = (Cat)((ObjectMessage) message).getObject();
                System.out.println(cat);
            } catch (JMSException e) {
                e.printStackTrace();
            }
        }
    }
}
~~~



### 消息超时/过期

```
producer.setTimeToLive();
```

设置了消息超时的消息，消费端在超时后进入死信队列，无法在消费到此消息。



给消息设置一个超时时间 -> 死信队列 -> 拿出来 -> 重发



#### 死信

此类消息会进入到`ActiveMQ.DLQ`队列且不会自动清除，称为死信

此处有消息堆积的风险

#### 修改死信队列名称

```xml
			  <policyEntry queue="user" prioritizedMessages="true" >
				<deadLetterStrategy> 

					<individualDeadLetterStrategy   queuePrefix="DeadQ." useQueueForQueueMessages="true" /> 

				</deadLetterStrategy> 
			  </policyEntry>
```

useQueueForQueueMessages: 设置使用队列保存死信，还可以设置useQueueForTopicMessages，使用Topic来保存死信 

#### 让非持久化的消息也进入死信队列

> 非持久化的消息不会存入数据库，超时以后自动被消费，不会进入死信队列
> 配置这项以后才会进死信队列

```xml
			<individualDeadLetterStrategy   queuePrefix="DLxxQ." useQueueForQueueMessages="true"  processNonPersistent="true" /> 
```

processNonPersistent="true"

#### 过期消息不进死信队列

```
<individualDeadLetterStrategy   processExpired="false"  /> 
```

### 独占消费者

> receiver配置后会优先消费消息，并且会霸占这个destination

```
	Queue queue = session.createQueue("xxoo?consumer.exclusive=true");
```

还可以设置优先级

```
Queue queue = session.createQueue("xxoo?consumer.exclusive=true&consumer.priority=10");
```

### 延迟消息投递

首先在配置文件中开启延迟和调度

**schedulerSupport="true"**

```
    <broker xmlns="http://activemq.apache.org/schema/core" brokerName="localhost" dataDirectory="${activemq.data}" schedulerSupport="true">
```



### 带间隔的重复发送	

如果你是在一个for循环里发送，activemq重复的是整个循环的信息，而不是单独重复每一个

repeat必须是int！

```java 
		long delay = 10 * 1000;	//延迟时间
		long period = 2 * 1000;	//额外发送周期
		int repeat = 9;		//额外重复9次，注意这个必须是int!!!
		message.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_DELAY, delay);
		message.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_PERIOD, period);
		message.setIntProperty(ScheduledMessage.AMQ_SCHEDULED_REPEAT, repeat);
		createProducer.send(message);
```



### 消息过滤

#### 分组过滤

**sender**

```java 
TextMessage textMessage = session.createTextMessage("hi: " + i);
textMessage.setLongProperty("group",i%3);
```

**receiver**

> 只接收 i%3 == 1 的消息

```java 
MessageConsumer consumer = session.createConsumer(queue,"group=1");
```

#### selector过滤

> selector只能选择header中的内容，而无法判断body的内容

**sender**

```java 
		MapMessage msg1 = session.createMapMessage();
	//这里是设置body信息	
		msg1.setString("name", "qiqi");
		msg1.setString("age", "18");
		
	//property才是header中的信息
		msg1.setStringProperty("name", "qiqi");
		msg1.setIntProperty("age", 18);
		MapMessage msg2 = session.createMapMessage();
		msg2.setString("name", "lucy");
		msg2.setString("age", "18");
		msg2.setStringProperty("name", "lucy");
		msg2.setIntProperty("age", 18);
		MapMessage msg3 = session.createMapMessage();
		msg3.setString("name", "qianqian");
		msg3.setString("age", "17");
		msg3.setStringProperty("name", "qianqian");
		msg3.setIntProperty("age", 17);
```

**receiver**

> 只接收 i%3 == 1 的消息

```java 
		String selector1 = "age > 17";
		String selector2 = "name = 'lucy'";
		MessageConsumer consumer = session.createConsumer(queue,selector2);
```

#### 

## 整合springBoot

#### 配置类

```java
@Configuration
@EnableJms
public class ActiveMqConfig {

    @Bean
    public JmsListenerContainerFactory<?> jmsListenerContainerTopic(ConnectionFactory activeMQConnectionFactory) {
        DefaultJmsListenerContainerFactory bean = new DefaultJmsListenerContainerFactory();
        bean.setPubSubDomain(true);
        bean.setConnectionFactory(activeMQConnectionFactory);
        return bean;
    }
    // queue模式的ListenerContainer
    @Bean
    public JmsListenerContainerFactory<?> jmsListenerContainerQueue(ConnectionFactory activeMQConnectionFactory) {
        DefaultJmsListenerContainerFactory bean = new DefaultJmsListenerContainerFactory();
        bean.setConnectionFactory(activeMQConnectionFactory);
        return bean;
    }
}
```

#### 发送端

JmsMessagingTemplate是简化版，没法配置各种选项

```java
@Service
public class JmsService {

    @Autowired
    private JmsTemplate jmsTemplate;
    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;
    //简化版
    public void send2(String destination,String msg) {
//        设置你想要的，如超时时间，持久化，事务啥的。
//        jmsTemplate.set
        List list = new ArrayList<>();
        list.add("you");
        list.add("are");
        list.add("sb");
        ActiveMQQueue queue = new ActiveMQQueue(destination);
        jmsMessagingTemplate.convertAndSend(queue,list);
    }
    
    

    public void send(String destination,String msg) {
//        设置你想要的，如超时时间，持久化，事务啥的。
//        jmsTemplate.set
        jmsTemplate.send(destination,new MessageCreator() {
            @Override
            public Message createMessage(Session session) throws JMSException {
                TextMessage textMessage = session.createTextMessage(msg);
                long delay = 10 * 1000;    //延迟时间
                long period = 2 * 1000;    //额外发送周期
                int repeat = 9;       //额外重复9次，注意这个必须是int!!!
                textMessage.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_DELAY, delay);
                textMessage.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_PERIOD, period);
                textMessage.setIntProperty(ScheduledMessage.AMQ_SCHEDULED_REPEAT, repeat);              
                return textMessage;
            }
        });
    }

}

```

#### 接收端

```java
@Component
public class JmsReceiver {


    @JmsListener(destination = "boot",containerFactory = "jmsListenerContainerQueue")
    public void receiveStringQueue(String msg) {
        System.out.println("接收到消息...." + msg);
    }

    @JmsListener(destination = "boot",containerFactory = "jmsListenerContainerTopic")
    public void receiveStringTopic(String msg) {
        System.out.println("接收到消息...." + msg);
    }
}
```



## Linux下

在`init.d`下建立软连接

```
cd /etc/init.d/
ln -s /usr/local/activemq/bin/activemq ./
```

**设置开启启动**

`chkconfig activemq on`

服务管理

```
service activemq start
service activemq status
service activemq stop
jps
```

客户端和用户端可以使用不同的传输协议。



Auto + Nio

```
<transportConnector name="auto+nio" uri="auto+nio://localhost:5671"/>
```

自动适配协议



## 面试题

### ActiveMQ如何防止消息丢失？会不会丢消息？

首先你的消息应该是持久化的，默认就是
日志复原

消息做了负载，有的receive不到

消息是有上限的

独占消费者导致



做高可用

死信队列、

持久化

ack

消息重投

记录日志

接收（消费）确认

### 产生100条消息再broker里，consumer接收到50的时候宕机了，重启后能准确的从51开始吗

这个消息肯定是在queue中的

首先看有没有事务，consumer接受后有没有ack, ack后有没有提交事务

然后看消息有没有做持久化，没做持久化的肯定就丢了。然后要看消息有没有设置优先级，有没有设置消息分组



### 如何防止重复消费？

消息幂等处理

map *ConcurrentHashMap* -> putIfAbsent   guava cache

### 如何保证消费顺序？

queue 优先级别设置

多消费端 -> 