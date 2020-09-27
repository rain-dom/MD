### Java主流框架演变之路

​		1、JSP+Servlet+JavaBean

​		2、MVC三层架构

![MVC](E:\deng\image\MVC.png)



![spring-overview](E:\deng\spring\spring-framework-5.2.3.RELEASE\docs\spring-framework-reference\images\spring-overview.png)





# IOC

service层不主动New对象，而是交给test来提供请求后再new对象

## 基本使用

### 1.maven构建

* **添加pom依赖**

pom.xml

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.mashibing</groupId>
    <artifactId>spring_demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.3.RELEASE</version>
        </dependency>
    </dependencies>
</project>
~~~

ioc.xml

~~~xml
//resource下的ioc.xml文件

<!-- xml name space -->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	
    <bean id="person" class="com.dzp.bean.Person">
        <property name="id" value="1"></property>
        <property name="gender" value="女"></property>
        <property name="age" value="20"></property>
        <property name="name" value="sakula"></property>
        <property name="date" value="2020/02/08"></property>
    </bean>
    
</beans>
~~~

test

~~~java
public class SpringDemoTest {
    public static void main(String[] args) {
        //ApplicationContext:表示ioc容器
        //ClassPathXmlApplicationContext:表示从当前classpath路径中获取xml文件的配置
        //根据spring的配置文件来获取ioc容器对象
        ApplicationContext context = new ClassPathXmlApplicationContext("ioc.xml");
        
        //上一步时已经创建了Person对象，       
        //括号内加了class，可以避免强制类型转换      
        context.getBean("person", Person.class);
        //通过类创建，但当Person多时报错
        context.getBean(Person.class);

        
        
        System.out.println(person);
    }
}
~~~

### 2、spring对象的获取及属性赋值方式

test

~~~java
		//括号内加了class，可以避免强制类型转换      
        context.getBean("person", Person.class);
        //通过类创建，但当Person多时报错
        context.getBean(Person.class);
~~~

#### **3、通过构造器给bean对象赋值**

ioc.xml

* 使用构造器赋值时，由构造方法的参数名称决定 

~~~xml
	<!--给person类添加构造方法-->
	<bean id="person2" class="com.mashibing.bean.Person">
        <constructor-arg name="id" value="1"></constructor-arg>
        <constructor-arg name="name" value="lisi"></constructor-arg>
        <constructor-arg name="age" value="20"></constructor-arg>
        <constructor-arg name="gender" value="女"></constructor-arg>
        
    </bean>

	<!--在使用构造器赋值时可省略name属性，但必须严格按照构造器参数的顺序来填写-->
	<bean id="person3" class="com.mashibing.bean.Person">
        <constructor-arg value="1"></constructor-arg>
        <constructor-arg value="lisi"></constructor-arg>
        <constructor-arg value="20"></constructor-arg>
        <constructor-arg value="女"></constructor-arg>
    </bean>

	<!--如果想不按照顺序来添加参数值，那么可以搭配index属性来使用-->
    <bean id="person4" class="com.mashibing.bean.Person">
        <constructor-arg value="lisi" index="1"></constructor-arg>
        <constructor-arg value="1" index="0"></constructor-arg>
        <constructor-arg value="女" index="3"></constructor-arg>
        <constructor-arg value="20" index="2"></constructor-arg>
    </bean>
	<!--当有多个参数个数相同，不同类型的构造器的时候，可以通过type来强制类型让系统知道你想要配置的是什么类型-->
	将person的age类型设置为Integer类型
	public Person(int id, String name, Integer age) {
        this.id = id;
        this.name = name;
        this.age = age;
        System.out.println("Age");
    }

    public Person(int id, String name, String gender) {
        this.id = id;
        this.name = name;
        this.gender = gender;
        System.out.println("gender");
    }
	<bean id="person5" class="com.mashibing.bean.Person">
        <constructor-arg value="1"></constructor-arg>
        <constructor-arg value="lisi"></constructor-arg>
        <constructor-arg value="20" type="java.lang.Integer"></constructor-arg>
    </bean>
	<!--如果不修改为integer类型，那么需要type跟index组合使用-->
	 <bean id="person5" class="com.mashibing.bean.Person">
        <constructor-arg value="1"></constructor-arg>
        <constructor-arg value="lisi"></constructor-arg>
        <constructor-arg value="20" type="int" index="2"></constructor-arg>
    </bean>
~~~

#### --4、通过命名空间为bean赋值，简化配置文件中属性声明的写法

​	1、导入命名空间

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

~~~

​	2、添加配置

~~~xml
    <bean id="person6" class="com.mashibing.bean.Person" p:id="3" p:name="wangwu" p:age="22" p:gender="男"></bean>
~~~

#### **5、为复杂类型进行赋值操作**

~~~xml
	<bean id="person" class="com.dzp.bean.Person">
        <property name="id" value="1"></prope rty>
        <property name="gender" value="女"></property>
        <property name="age" value="20"></property>
        <property name="name" value="sakula"></property>
    </bean>
    <bean id="person2" class="com.dzp.bean.Person" p:id="3" p:name="lisa" p:age="10" p:gender="女"></bean>

    <bean id="book" class="com.dzp.bean.Book">
        <property name="name" value="百年孤独"></property>
        <property name="author" value="鲁迅"></property>
        <property name="price" value="100"></property>
    </bean>



    <bean id="person3" class="com.dzp.bean.Person">
        <property name="book" ref="book" ></property>
        
        <property name="hobbies" value="book,girl,football"></property>
        
        <property name="lists">
            <list>
<!--内部bean,无法从IOC容器中直接获取-->
                <bean class="com.dzp.bean.Book">
                    <property name="price" value="10"></property>
                    <property name="author" value="张三"></property>
                    <property name="name" value="黑鹰坠落"></property>
                </bean>
                <ref bean="book"></ref>
            </list>
        </property>
        
        <property name="sets">
            <set>
                <value>1</value>
                <value>2</value>
            </set>
        </property>
        
        <property name="maps">
            <map>
                <entry key="a" value="12"></entry>
                <entry key="loveBook" value-ref="book"></entry>
                <entry key="book2">
                    <bean class="com.dzp.bean.Book">
                        <property name="name" value="职场尤里卡"></property>
                    </bean>
                </entry>

            </map>
        </property>

    </bean>

    <bean id="person4" class="com.dzp.bean.Person">
        <property name="id" value="1"></property>
<!--        给数组赋值-->
        <property name="hobbies">
            <array>
                <value>book</value>
                <value>girl</value>
                <value>football</value>
            </array>
        </property>
    </bean>
~~~



#### ==6、继承关系bean的配置

ioc.xml

~~~xml
   	<!--模拟抽象类，加入关键字abstract-->
	<bean id="father" class="com.dzp.bean.Person" abstract="true">
        <property name="id" value="1"></property>
        <property name="name" value="sakula"></property>
    </bean>
    <!--parent:指定bean的配置信息继承于哪个bean,子类重复信息覆盖父类-->
    <bean id="son" class="com.dzp.bean.Person" parent="father">
        <property name="name" value="xue"></property>
    </bean>
~~~

#### 7、bean对象创建的依赖关系

​		bean对象在创建的时候是按照bean在配置文件的**顺序**决定的，也可以使用depend-on标签来决定顺序

ioc.xml

~~~xml
	<bean id="book" class="com.mashibing.bean.Book" depends-on="person,address"></bean>
    <bean id="address" class="com.mashibing.bean.Address"></bean>
    <bean id="person" class="com.mashibing.bean.Person"></bean>
~~~

#### ==8、bean的作用域控制，是否是单例

~~~xml
    <!--
    bean的作用域：singleton、prototype、request、session
    默认情况下是单例的
    prototype：多实例的
        容器启动的时候不会创建多实例bean，只有在获取对象的时候才会创建该对象
        每次创建都是一个新的对象
    singleton：默认的单例对象
        在容器启动完成之前就已经创建好对象
        获取的所有对象都是同一个
    -->
    <bean id="person4" class="com.mashibing.bean.Person" scope="prototype"></bean>
~~~

#### 9、利用工厂模式创建bean对象

静态工厂：工厂本身不需要创建对象，但是可以通过静态方法调用，对象=工厂类.静态工厂方法名（）；

* 获取对象的方法必须是static的，否则xml无法读取

PersonStaticFactory.java

~~~java
package com.mashibing.factory;

import com.mashibing.bean.Person;

public class PersonStaticFactory {

    public static Person getPerson(String name){
        Person person = new Person();
        person.setId(1);
        person.setName(name);
        return person;
    }
}
~~~

ioc.xml

~~~xml
<!--
静态工厂的使用：
class:指定静态工厂类
factory-method:指定哪个方法是工厂方法
-->
	<bean id="person5" class="com.mashibing.factory.PersonStaticFactory" factory-method="getPerson">
        <!--constructor-arg：可以为方法指定参数-->
        <constructor-arg value="lisi"></constructor-arg>
    </bean>
~~~



实例工厂：工厂本身需要创建对象（**加入bean**），工厂类 工厂对象=new 工厂类；工厂对象.get对象名（）；

* 方法不必是静态的

PersonInstanceFactory.java

~~~java
package com.mashibing.factory; 

import com.mashibing.bean.Person;

public class PersonInstanceFactory {
    public Person getPerson(String name){
        Person person = new Person();
        person.setId(1);
        person.setName(name);
        return person;
    }
}
~~~

ioc.xml

~~~xml
    <!--实例工厂使用-->
    <!--创建实例工厂类-->
    <bean id="personInstanceFactory" class="com.mashibing.factory.PersonInstanceFactory"></bean>
    <!--
    factory-bean:指定使用哪个工厂实例
    factory-method:指定使用哪个工厂实例的方法
    -->
    <bean id="person6" class="com.mashibing.bean.Person" factory-bean="personInstanceFactory" factory-method="getPerson">
        <constructor-arg value="wangwu"></constructor-arg>
    </bean>
~~~

#### ==10、继承FactoryBean来创建对象

FactoryBean是Spring规定的一个接口，当前接口的实现类，Spring都会将其作为一个工厂，但是**在ioc容器启动的时候不会创建实例**，只有在使用的时候才会创建对象

MyFactoryBean.java

~~~java
package com.mashibing.factory;

import com.mashibing.bean.Person;
import org.springframework.beans.factory.FactoryBean;

/**
 * 实现了FactoryBean接口的类是Spring中可以识别的工厂类，spring会自动调用工厂方法创建实例
 */

public class MyFactoryBean implements FactoryBean<Person> {

    /**
     * 工厂方法，返回需要创建的对象
     * @return
     * @throws Exception
     */
    @Override
    public Person getObject() throws Exception {
        Person person = new Person();
        person.setName("maliu");
        return person;
    }

    /**
     * 返回创建对象的类型,spring会自动调用该方法返回对象的类型
     * @return
     */
    @Override
    public Class<?> getObjectType() {
        return Person.class;
    }

    /**
     * 创建的对象是否是单例对象
     * @return
     */
    @Override
    public boolean isSingleton() {
        return false;
    }
}
~~~

ioc.xml

~~~xml
<bean id="myfactorybean" class="com.mashibing.factory.MyFactoryBean"></bean>

<!--Test中-->
Person myFactoryBean = context.getBean("myFactory",Person.class);
        System.out.println(myFactoryBean);
~~~

#### ==11、bean对象的初始化和销毁方法

​		在创建对象的时候，我们可以根据需要调用初始化和销毁的方法

Person.java

~~~java

    public void init(){
        System.out.println("对象被初始化");
    }
    
    public void destory(){
        System.out.println("对象被销毁");
    }
    
~~~

ioc.xml

~~~xml
<!--bean生命周期表示bean的创建到销毁
        如果scope是singleton(单例)，容器在启动的时候会创建好，关闭的时候会销毁创建的bean
        如果scope是prototype(多例)，获取的时候创建对象，销毁的时候不会有任何的调用
    -->
    <bean id="address" class="com.mashibing.bean.Address" init-method="init" destroy-method="destory"></bean>
~~~

MyTest.java

~~~java
import com.mashibing.bean.Address;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MyTest {
    public static void main(String[] args) {
        //这里context是ApplicationContext类下，无close方法
        ApplicationContext context = new ClassPathXmlApplicationContext("ioc.xml");
        Address address = context.getBean("address", Address.class);
        System.out.println(address);
        //applicationContext没有close方法，需要使用具体的子类
        ((ClassPathXmlApplicationContext)context).close();

    }
}
~~~

#### ？？12、配置bean对象初始化方法的前后处理方法

​		spring中包含一个BeanPostProcessor的接口，可以在bean的初始化方法的前后调用该方法，如果配置了初始化方法的前置和后置处理器，无论是否包含初始化方法，都会进行调用

MyBeanPostProcessor.java

~~~java
package com.mashibing.bean;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

public class MyBeanPostProcessor implements BeanPostProcessor {
    /**
     * 在初始化方法调用之前执行
     * @param bean  初始化的bean对象
     * @param beanName  xml配置文件中的bean的id属性
     * @return
     * @throws BeansException
     */
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization:"+beanName+"调用初始化前置方法");
        return bean;
    }

    /**
     * 在初始化方法调用之后执行
     * @param bean
     * @param beanName
     * @return
     * @throws BeansException
     */
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization:"+beanName+"调用初始化后缀方法");
        return bean;
    }
}
~~~

ioc.xml

~~~xml
<bean id="myBeanPostProcessor" class="com.mashibing.bean.MyBeanPostProcessor"></bean>
~~~

### 3、spring创建第三方bean对象

在Spring中，很多对象都是单实例的，在日常的开发中，我们经常需要使用某些外部的单实例对象，例如数据库连接池，下面即如何在spring中创建第三方bean实例。

​	

​	1、导入数据库连接池的pom文件

* 从maven仓库复制dependence

~~~xml
<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.21</version>
</dependency>
<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.47</version>
</dependency>
~~~

​		2、编写配置文件

ioc.xml

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="username" value="root"></property>
        <property name="password" value="123456"></property>
        <property name="url" value="jdbc:mysql://localhost:3306/dzp1"></property>
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    </bean>
</beans>
~~~

​		3、编写测试文件

MyTest.java

~~~java
import com.alibaba.druid.pool.DruidDataSource;
import com.mashibing.bean.Address;
import com.mashibing.bean.Person;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import java.sql.SQLException;

public class MyTest {
    public static void main(String[] args) throws SQLException {
        ApplicationContext context = new ClassPathXmlApplicationContext("ioc3.xml");
        DruidDataSource dataSource = context.getBean("dataSource", DruidDataSource.class);
        System.out.println(dataSource);
        System.out.println(dataSource.getConnection());
    }
}
~~~

### ==4、spring引用外部配置文件

在resource中添加dbconfig.properties

~~~properties
username=root
password=123456
url=jdbc:mysql://localhost:3306/demo
driverClassName=com.mysql.jdbc.Driver
~~~

编写配置文件

~~~xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
下面是
       xmlns:context="http://www.springframework.org/schema/context"

       xmlns:p="http://www.springframework.org/schema/p"

       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
下面2个是配置
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
">
        
	<!--加载外部配置文件
		在加载外部依赖文件的时候需要context命名空间
	-->  
    <context:property-placeholder location="classpath:dbconfig.properties"/>
    
    <bean id="dataSource2" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="username" value="${username}"></property>
        <property name="password" value="${password}"></property>
        <property name="url" value="${url}"></property>
        <property name="driverClassName" value="${driverClassName}"></property>
    </bean>
</beans>
    
    
~~~



### 5、spring基于xml文件的自动装配

​		**当一个对象中需要引用另外一个对象的时候**，在之前的配置中我们都是通过property标签来进行手动配置的，其实在spring中还提供了一个非常强大的功能就是自动装配，**如将bean中的address自动装配到person中**可以按照我们指定的规则进行配置，配置的方式有以下几种：

​		default/no：不自动装配

​		byName：按照名字进行装配，以属性名作为id去容器中查找组件，用**set方法**进行赋值，如果找不到则装配null

​		byType：按照类型进行装配,以属性的类型作为查找依据去容器中找到这个组件，如果有多个类型相同的bean对象，那么会报异常，如果找不到则装配null

​		constructor：按照构造器进行装配，首先按照类型判断，如果有多个类型相同的bean,再按照id进行判断

先按照有参构造器参数的类型进行装配，没有就直接装配null；如果按照类型找到了多个，那么就使用参数名作为id继续匹配，找到就装配，找不到就装配null

ioc.xml

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="address" class="com.mashibing.bean.Address">
        <property name="province" value="河北"></property>
        <property name="city" value="邯郸"></property>
        <property name="town" value="武安"></property>
    </bean>
    <bean id="person" class="com.mashibing.bean.Person" autowire="byName"></bean>
    <bean id="person2" class="com.mashibing.bean.Person" autowire="byType"></bean>
    <bean id="person3" class="com.mashibing.bean.Person" autowire="constructor"></bean>
</beans>
~~~

### *6、SpEL的使用

​		SpEL:Spring Expression Language,spring的表达式语言，支持运行时查询操作对象

​		使用#{...}作为语法规则，所有的大括号中的字符都认为是SpEL.

ioc.xml

```xml
    <bean id="person4" class="com.mashibing.bean.Person">
        <!--支持任何运算符-->
        <property name="age" value="#{12*2}"></property>
        <!--可以引用其他bean的某个属性值-->
        <property name="name" value="#{address.province}"></property>
        <!--引用其他bean-->
        <property name="address" value="#{address}"></property>
        <!--调用静态方法-->
        <property name="hobbies" value="#{T(java.util.UUID).randomUUID().toString().substring(0,4)}"></property>
        <!--调用非静态方法-->
        <property name="gender" value="#{address.getCity()}"></property>
    </bean>
```



## 注解

在之前的项目中，我们都是通过xml文件进行bean或者某些属性的赋值，其实还有另外一种注解的方式，在企业开发中使用的很多，在bean上添加注解，可以快速的将bean注册到ioc容器。





### 1、使用注解的方式注册bean到IOC容器中



使用注解需要如下步骤：
1、添加上述四个注解中的任意一个
2、添加自动扫描注解的组件，此操作需要依赖context命名空间
3、添加自动扫描的标签context:component-scan

注意：当使用注解注册组件和使用配置文件注册组件是一样的，但是要注意：
	1、组件的id默认就是组件的类名首字符**小写**，如果非要改名字的话，直接在注解中添加即可

​	2、组件默认情况下都是**单例**的,如果需要配置多例模式的话，可以在注解下添加**@Scope**注解



applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
记得添加context
       xmlns:context="http://www.springframework.org/schema/context"
       
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
                           
下面这两条要自己导入                   
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

    <!--
    如果想要将自定义的bean对象添加到IOC容器中，需要在类上添加某些注解
    Spring中包含4个主要的组件添加注解：
    @Controller:控制器，推荐给controller层添加此注解
    @Service:业务逻辑，推荐给业务逻辑层添加此注解
    @Repository:仓库管理，推荐给数据访问层添加此注解
    @Component:给不属于以上基层的组件添加此注解
    注意：我们虽然人为的给不同的层添加不同的注解，但是在spring看来，可以在任意层添加任意注解
           spring底层是不会给具体的层次验证注解，这样写的目的只是为了提高可读性

    使用注解需要如下步骤：
    1、添加上述四个注解中的任意一个
    2、添加自动扫描注解的组件，此操作需要依赖context命名空间
    3、添加自动扫描的标签context:component-scan

	注意：当使用注解注册组件和使用配置文件注册组件是一样的，但是要注意：
		1、组件的id默认就是组件的类名首字符小写，如果非要改名字的话，直接在注解中添加即可

		2、组件默认情况下都是单例的,如果需要配置多例模式的话，可以在注解下添加@Scope注解
    -->
    <!--
    定义自动扫描的基础包:
    base-package:指定扫描的基础包，spring在启动的时候会将基础包及子包下所有加了注解的类都自动
                扫描进IOC容器
    -->
    <context:component-scan base-package="com.mashibing"></context:component-scan>
</beans>
```

PersonController.java

```java
package com.mashibing.controller;

import org.springframework.stereotype.Controller;

@Controller
public class PersonController {
    public PersonController() {
        System.out.println("创建对象");
    }
}
```

PersonService.java

```java
package com.mashibing.service;

import org.springframework.stereotype.Service;

@Service
public class PersonService {
}
```

PersonDao.java

```java
package com.mashibing.dao;

import org.springframework.stereotype.Repository;

@Repository("personDao")
@Scope(value="prototype")
public class PersonDao {
}
```

### 2、定义扫描包时要包含的类和不要包含的类

​		当定义好基础的扫描包后，在某些情况下可能要有选择性的配置是否要注册bean到IOC容器中，此时可以通过如下的方式进行配置。

自动调用构造方法

applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
    
    <!--指定只扫描哪些组件，默认情况下是全部扫描的，所以此时要配置的话需要在component-scan标签中添加 use-default-filters="false"-->
    <context:component-scan base-package="com.mashibing" use-default-filters="false">
        <!--
        当定义好基础扫描的包之后，可以排除包中的某些类，使用如下的方式:
        type:表示指定过滤的规则
            annotation：按照注解进行排除，标注了指定注解的组件不要,@component下的包内完全限定名
            assignable：指定排除某个具体的类，按照类排除，expression表示不注册的具体类名
            aspectj：后面讲aop的时候说明要使用的aspectj表达式，不用
            custom：定义一个typeFilter,自己写代码决定哪些类被过滤掉，不用
            regex：使用正则表达式过滤，不用
        -->
 <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>

        
        <context:include-filter type="assignable" expression="com.mashibing.service.PersonService"/>
    </context:component-scan>
</beans>
```

### 3、使用@AutoWired进行自动注入

​		使用注解的方式实现自动注入需要使用@AutoWired注解。

PersonController.java

```java
package com.mashibing.controller;

import com.mashibing.service.PersonService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;

@Controller
public class PersonController {
	
    //需要自动注入对象，否则因为没有具体对象而无法调用其方法而报错
    @Autowired
    private PersonService personService;

    public PersonController() {
        System.out.println("创建对象");
    }

    public void getPerson(){
        personService.getPerson();
    }
}
```

PersonService.java

```java
package com.mashibing.service;

import com.mashibing.dao.PersonDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class PersonService {

    @Autowired
    private PersonDao personDao;

    public void getPerson(){
        personDao.getPerson();
    }
}
```

PersonDao.java

```java
package com.mashibing.dao;

        import org.springframework.stereotype.Repository;

@Repository
public class PersonDao {

    public void getPerson(){
        System.out.println("PersonDao:getPerson");
    }
}
```

注意：当使用AutoWired注解的时候，自动装配的时候是根据**类型**实现的。

​		1、如果只找到一个，则直接进行赋值，

​		2、如果没有找到，则直接抛出异常，

​		3、如果找到多个，那么会按照变量名作为id继续匹配,

​				1、匹配上直接进行装配

​				2、如果匹配不上则直接报异常

PersonServiceExt.java

```java
package com.mashibing.service;

import com.mashibing.dao.PersonDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class PersonServiceExt extends PersonService{

    @Autowired
    private PersonDao personDao;

    public void getPerson(){
        System.out.println("PersonServiceExt......");
        personDao.getPerson();
    }
}
```

PersonController.java

```java
package com.mashibing.controller;

import com.mashibing.service.PersonService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;

@Controller
public class PersonController {

    @Autowired
    private PersonService personServiceExt;

    public PersonController() {
        System.out.println("创建对象");
    }

    public void getPerson(){
        personServiceExt.getPerson();
    }
}
```

​		还可以使用@Qualifier注解来指定id的名称，让spring不要使用变量名,当使用@Qualifier注解的时候也会有两种情况：

​		1、找到，则直接装配

​		2、找不到，就会报错

PersonController.java

```java
package com.mashibing.controller;

import com.mashibing.service.PersonService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Controller;

@Controller
public class PersonController {

    @Autowired
    @Qualifier("personService")
    private PersonService personServiceExt2;

    public PersonController() {
        System.out.println("创建对象");
    }

    public void getPerson(){
        personServiceExt2.getPerson();
    }
}
```

​		通过上述的代码我们能够发现，使用@AutoWired肯定是能够装配上的，如果装配不上就会报错。

### xx4、@AutoWired可以进行定义在方法上和@qualifier

​		当我们查看@AutoWired注解的源码的时候发现，此注解不仅可以使用在成员变量上，也可以使用在方法上。

PersonController.java

```java
package com.mashibing.controller;

import com.mashibing.dao.PersonDao;
import com.mashibing.service.PersonService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Controller;

@Controller
public class PersonController {

    @Qualifier("personService")
    @Autowired
    private PersonService personServiceExt2;

    public PersonController() {
        System.out.println("创建对象");
    }

    public void getPerson(){
        System.out.println("personController..."+personServiceExt2);
//        personServiceExt2.getPerson();
    }

     /**
     * 当方法上有@AutoWired注解时：
     *  1、此方法在bean创建的时候会自动调用
     *  2、这个方法的每一个参数都会自动注入值
     * @param personDao
     */
    @Autowired
    public void test(PersonDao personDao){
        System.out.println("此方法被调用:"+personDao);
    }
    
    /**
     * @Qualifier注解也可以作用在属性上，用来被当作id去匹配容器中的对象，如果没有
     * 此注解，那么直接按照类型进行匹配
     * @param personService
     */
    @Autowired
    public void test2(@Qualifier("personServiceExt") PersonService personService){
        System.out.println("此方法被调用："+personService);
    }
}
```

### 5、自动装配的注解@AutoWired，@Resource

​		在使用自动装配的时候，出了可以使用@AutoWired注解之外，还可以使用@Resource注解，大家需要知道这两个注解的区别。

​		1、@AutoWired:是spring中提供的注解，@Resource:是jdk中定义的注解，依靠的是java的标准

​		2、@AutoWired默认是按照类型进行装配，找不到则通过名字查找，默认情况下要求依赖的对象必须存在，@Resource默认是按照名字进行匹配的，同时可以指定name属性。

​		3、@AutoWired只适合spring框架，而@Resource扩展性更好

PersonController.java

```java
package com.mashibing.controller;

import com.mashibing.dao.PersonDao;
import com.mashibing.service.PersonService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Controller;

import javax.annotation.Resource;

@Controller
public class PersonController {

    @Qualifier("personService")
    @Resource
    private PersonService personServiceExt2;

    public PersonController() {
        System.out.println("创建对象");
    }

    public void getPerson(){
        System.out.println("personController..."+personServiceExt2);
        personServiceExt2.getPerson();
    }

    /**
     * 当方法上有@AutoWired注解时：
     *  1、此方法在bean创建的时候会自动调用
     *  2、这个方法的每一个参数都会自动注入值
     * @param personDao
     */
    @Autowired
    public void test(PersonDao personDao){
        System.out.println("此方法被调用:"+personDao);
    }

    /**
     * @Qualifier注解也可以作用在属性上，用来被当作id去匹配容器中的对象，如果没有
     * 此注解，那么直接按照类型进行匹配
     * @param personService
     */
    @Autowired
    public void test2(@Qualifier("personServiceExt") PersonService personService){
        System.out.println("此方法被调用："+personService);
    }
}
```

### 6、泛型依赖注入

​		为了讲解泛型依赖注入，首先我们需要先写一个基本的案例，按照我们之前学习的知识：

Student.java

```java
package com.mashibing.bean;

public class Student {
}
```

Teacher.java

```java
package com.mashibing.bean;

public class Teacher {
}
```

BaseDao.java

```java
package com.mashibing.dao;

import org.springframework.stereotype.Repository;

@Repository
public abstract class BaseDao<T> {

    public abstract void save();
}
```

StudentDao.java

```java
package com.mashibing.dao;

import com.mashibing.bean.Student;
import org.springframework.stereotype.Repository;

@Repository
public class StudentDao extends BaseDao<Student>{
    public void save() {
        System.out.println("保存学生");
    }
}
```

TeacherDao.java

```java
package com.mashibing.dao;

import com.mashibing.bean.Teacher;
import org.springframework.stereotype.Repository;

@Repository
public class TeacherDao extends BaseDao<Teacher> {
    public void save() {
        System.out.println("保存老师");
    }
}
```

StudentService.java

```java
package com.mashibing.service;

import com.mashibing.dao.StudentDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class StudentService {

    @Autowired
    private StudentDao studentDao;

    public void save(){
        studentDao.save();
    }
}
```

TeacherService.java

```java
package com.mashibing.service;

import com.mashibing.dao.TeacherDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class TeacherService {
    @Autowired
    private TeacherDao teacherDao;

    public void save(){
        teacherDao.save();
    }
}
```

MyTest.java

```java
import com.mashibing.service.StudentService;
import com.mashibing.service.TeacherService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import javax.sql.DataSource;
import java.sql.SQLException;

public class MyTest {
    public static void main(String[] args) throws SQLException {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        StudentService studentService = context.getBean("studentService",StudentService.class);
        studentService.save();

        TeacherService teacherService = context.getBean("teacherService",TeacherService.class);
        teacherService.save();
    }
}
```

​		上述代码是我们之前的可以完成的功能，但是可以思考，Service层的代码是否能够改写：

BaseService不用注入，会与子类产生冲突

base中没有注入具体类型，而是使用泛型。在调用子类Service时会根据子类的泛型去匹配Dao中的泛型去执行方法



BaseService.java

```java
package com.mashibing.service;

import com.mashibing.dao.BaseDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

public class BaseService<T> {
    
    @Autowired
    BaseDao<T> baseDao;
    
    public void save(){
        System.out.println("自动注入的对象："+baseDao);
        baseDao.save();
    }
}
```



子类只是继承父类，没有多余功能，

StudentService.java

```java
package com.mashibing.service;

import com.mashibing.bean.Student;
import com.mashibing.dao.StudentDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class StudentService extends BaseService<Student> {

}
```

TeacherService.java

```java
package com.mashibing.service;

import com.mashibing.bean.Teacher;
import com.mashibing.dao.TeacherDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class TeacherService extends BaseService<Teacher>{

}
```



# AOP

## 简单使用

想要在程序运行过程中动态的获取方法的名称及参数、结果等相关信息，此时可以通过使用**动态代理**的方式来进行实现。

**麻烦的方式**

Calculator.java

```java
package com.mashibing.inter;

public interface Calculator {

    public int add(int i,int j);

    public int sub(int i,int j);

    public int mult(int i,int j);

    public int div(int i,int j);
}
```

MyCalculator.java

```java
package com.mashibing.inter;

public class MyCalculator implements Calculator {
    public int add(int i, int j) {
        int result = i + j;
        return result;
    }

    public int sub(int i, int j) {
        int result = i - j;
        return result;
    }

    public int mult(int i, int j) {
        int result = i * j;
        return result;
    }

    public int div(int i, int j) {
        int result = i / j;
        return result;
    }
}
```

MyTest.java

```java
public class MyTest {
    public static void main(String[] args) throws SQLException {
        MyCalculator myCalculator = new MyCalculator();
        System.out.println(myCalculator.add(1, 2));
    }
}
```

​		此代码非常简单，就是基础的javase的代码实现，此时如果需要添加日志功能应该怎么做呢，只能在每个方法中添加日志输出，同时如果需要修改的话会变得非常麻烦。

MyCalculator.java

```java
package com.mashibing.inter;

public class MyCalculator implements Calculator {
    public int add(int i, int j) {
        System.out.println("add 方法开始执行，参数为："+i+","+j);
        int result = i + j;
        System.out.println("add 方法开始完成结果为："+result);
        return result;
    }

    public int sub(int i, int j) {
        System.out.println("sub 方法开始执行，参数为："+i+","+j);
        int result = i - j;
        System.out.println("add 方法开始完成结果为："+result);
        return result;
    }

    public int mult(int i, int j) {
        System.out.println("mult 方法开始执行，参数为："+i+","+j);
        int result = i * j;
        System.out.println("add 方法开始完成结果为："+result);
        return result;
    }

    public int div(int i, int j) {
        System.out.println("div 方法开始执行，参数为："+i+","+j);
        int result = i / j;
        System.out.println("add 方法开始完成结果为："+result);
        return result;
    }
}
```

​		可以考虑将日志的处理抽象出来，变成工具类来进行实现：

LogUtil.java

public static void start(Object ... objects){}

```java
package com.mashibing.util;

import java.util.Arrays;

public class LogUtil {

    public static void start(Object ... objects){
        System.out.println("XXX方法开始执行，使用的参数是："+ Arrays.asList(objects));
    }

    public static void stop(Object ... objects){
        System.out.println("XXX方法执行结束，结果是："+ Arrays.asList(objects));
    }
}
```

MyCalculator.java

```java
package com.mashibing.inter;

import com.mashibing.util.LogUtil;

public class MyCalculator implements Calculator {
    public int add(int i, int j) {
        LogUtil.start(i,j);
        int result = i + j;
        LogUtil.stop(result);
        return result;
    }

    public int sub(int i, int j) {
        LogUtil.start(i,j);
        int result = i - j;
        LogUtil.stop(result);
        return result;
    }

    public int mult(int i, int j) {
        LogUtil.start(i,j);
        int result = i * j;
        LogUtil.stop(result);
        return result;
    }

    public int div(int i, int j) {
        LogUtil.start(i,j);
        int result = i / j;
        LogUtil.stop(result);
        return result;
    }
}
```

​		按照上述方式抽象之后，

#### 动态代理

该已经发现在输出的信息中并不包含具体的方法名称，我们更多的是想要在程序运行过程中动态的获取方法的名称及参数、结果等相关信息，此时可以通过使用**动态代理**的方式来进行实现。



CalculatorProxy.java

将自己的calculator传入，输出代理的calculator(变成proxy)

Proxy.newProxyInstance(loader, interfaces, h)

将想要代理的类的信息导入

~~~JAVA
package com.mashibing.proxy;

import com.mashibing.inter.Calculator;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.Arrays;

/**
 * 帮助Calculator生成代理对象的类
 */
public class CalculatorProxy {

    /**
     *
     *  为传入的参数对象创建一个动态代理对象
     * @param calculator 被代理对象
     * @return
     */
    public static Calculator getProxy(final Calculator calculator){


        //获取被代理对象的类加载器
        ClassLoader loader = calculator.getClass().getClassLoader();
        //获取被代理对象的所有接口
        Class<?>[] interfaces = calculator.getClass().getInterfaces();
        //被代理类想要执行的方法
        InvocationHandler h = new InvocationHandler() {
            /**
             *  执行目标方法
             * @param proxy 代理对象，给jdk使用，任何时候都不要操作此对象
             * @param method 当前将要执行的目标对象的方法
             * @param args 这个方法调用时外界传入的参数值
             * @return
             * @throws Throwable
             */
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                //利用反射执行目标方法,目标方法执行后的返回值
//                System.out.println("这是动态代理执行 = 方法");
                Object result = null;
                try {
                    LogUtil.start(method,args);
                    result = method.invoke(calculator, args);
                    LogUtil.stop(method,args);
                } catch (Exception e) {
                    LogUtil.logException(method,e);
                } finally {
                    LogUtil.end(method);
                }
                //将结果返回回去
                return result;
            }
        };
        Object proxy = Proxy.newProxyInstance(loader, interfaces, h);
        return (Calculator) proxy;
    }
}
~~~



LogUtil.java

这种方式更加灵活，而且不需要在业务方法中添加额外的代码，这才是常用的方式。还可以使用日志工具类来完善日志。

```java
package com.mashibing.util;

import java.lang.reflect.Method;
import java.util.Arrays;

public class LogUtil {

    public static void start(Method method, Object ... objects){
//        System.out.println("XXX方法开始执行，使用的参数是："+ Arrays.asList(objects));
        System.out.println(method.getName()+"方法开始执行，参数是："+ Arrays.asList(objects));
    }

    public static void stop(Method method,Object ... objects){
//        System.out.println("XXX方法执行结束，结果是："+ Arrays.asList(objects));
        System.out.println(method.getName()+"方法开始执行，参数是："+ Arrays.asList(objects));

    }

    public static void logException(Method method,Exception e){
        System.out.println(method.getName()+"方法出现异常："+ e.getMessage());
    }
    
    public static void end(Method method){
        System.out.println(method.getName()+"方法执行结束了......");
    }
}
```



这种动态代理的实现方式调用的是jdk的基本实现，如果需要代理的目标对象没有实现任何接口，那么是无法为他创建代理对象的，这也是致命的缺陷。而在Spring中我们可以不编写上述如此复杂的代码，只需要利用AOP，就能够轻轻松松实现上述功能，当然，Spring AOP的底层实现也依赖的是动态代理。

## 概念及术语

- 切面（Aspect）: 指关注点模块化，这个关注点可能会横切多个对象。事务管理是企业级Java应用中有关横切关注点的例子。 在Spring AOP中，切面可以使用通用类基于模式的方式（schema-based approach）或者在普通类中以`@Aspect`注解（@AspectJ 注解方式）来实现。
- 连接点（Join point）: 在程序执行过程中某个特定的点，例如某个方法调用的时间点或者处理异常的时间点。在Spring AOP中，一个连接点总是代表一个方法的执行。
- 通知（Advice）: 在切面的某个特定的连接点上执行的动作。通知有多种类型，包括“around”, “before” and “after”等等。通知的类型将在后面的章节进行讨论。 许多AOP框架，包括Spring在内，都是以拦截器做通知模型的，并维护着一个以连接点为中心的拦截器链。
- 切点（Pointcut）: 匹配连接点的断言。通知和切点表达式相关联，并在满足这个切点的连接点上运行（例如，当执行某个特定名称的方法时）。切点表达式如何和连接点匹配是AOP的核心：Spring默认使用AspectJ切点语义。
- 引入（Introduction）: 声明额外的方法或者某个类型的字段。Spring允许引入新的接口（以及一个对应的实现）到任何被通知的对象上。例如，可以使用引入来使bean实现 `IsModified`接口， 以便简化缓存机制（在AspectJ社区，引入也被称为内部类型声明（inter））。
- 目标对象（Target object）: 被一个或者多个切面所通知的对象。也被称作被通知（advised）对象。既然Spring AOP是通过运行时代理实现的，那么这个对象永远是一个被代理（proxied）的对象。
- AOP代理（AOP proxy）:AOP框架创建的对象，用来实现切面契约（aspect contract）（包括通知方法执行等功能）。在Spring中，AOP代理可以是JDK动态代理或CGLIB代理。
- 织入（Weaving）: 把切面连接到其它的应用程序类型或者对象上，并创建一个被被通知的对象的过程。这个过程可以在编译时（例如使用AspectJ编译器）、类加载时或运行时中完成。 Spring和其他纯Java AOP框架一样，是在运行时完成织入的。

##### AOP的通知类型

- 前置通知（Before advice）: 在连接点之前运行但无法阻止执行流程进入连接点的通知（除非它引发异常）。
- 后置返回通知（After returning advice）:在连接点正常完成后执行的通知（例如，当方法没有抛出任何异常并正常返回时）。
- 后置异常通知（After throwing advice）: 在方法抛出异常退出时执行的通知。
- 后置通知（总会执行）（After (finally) advice）: 当连接点退出的时候执行的通知（无论是正常返回还是异常退出）。
- 环绕通知（Around Advice）:环绕连接点的通知，例如方法调用。这是最强大的一种通知类型，。环绕通知可以在方法调用前后完成自定义的行为。它可以选择是否继续执行连接点或直接返回自定义的返回值又或抛出异常将执行结束。

##### AOP的应用场景

- 日志管理
- 权限认证
- 安全检查
- 事务控制

## Spring AOP的简单配置

### 1.添加pom依赖

​	spring AOP 、CGLib、AspectJ、AOP Alliance、spring aspects

### 2、编写配置

- 将目标类和切面类加入到IOC容器中，在对应的类上添加组件注解

  - 给LogUtil添加@Component注解

  - 给MyCalculator添加@Service注解

  - 添加自动扫描的配置

    ```xml
    <!--别忘了添加context命名空间-->
    <context:component-scan base-package="com.dzp"></context:component-scan>
     <!--开启aop的注解功能-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
    ```

- 设置程序中的切面类

  - 在LogUtil.java中添加@Aspect注解

- 设置切面类中的方法是什么时候在哪里执行

  - @Before:在目标方法之前运行：前置通知
  - @After:在目标方法之后运行：后置通知
  - @AfterReturning:在目标方法正常返回之后：返回通知
  - @AfterThrowing:在目标方法抛出异常后开始运行：异常通知
  - @Around:环绕：环绕通知



- 开启基于注解的aop的功能

- 设置切面类中的方法是什么时候在哪里执行

  ```java
  package com.mashibing.util;
  
  import org.aspectj.lang.annotation.*;
  import org.springframework.stereotype.Component;
  
  import java.lang.reflect.Method;
  import java.util.Arrays;
  
  @Component
  @Aspect
  public class LogUtil {
  
      /*
      设置下面方法在什么时候运行
          @Before:在目标方法之前运行：前置通知
          @After:在目标方法之后运行：后置通知
          @AfterReturning:在目标方法正常返回之后：返回通知
          @AfterThrowing:在目标方法抛出异常后开始运行：异常通知
          @Around:环绕：环绕通知
  
          当编写完注解之后还需要设置在哪些方法上执行，使用表达式
          execution(访问修饰符  返回值类型 方法全称)
       */
      @Before("execution( public int com.mashibing.inter.MyCalculator.*(int,int))")
      public static void start(){
  //        System.out.println("XXX方法开始执行，使用的参数是："+ Arrays.asList(objects));
  //        System.out.println(method.getName()+"方法开始执行，参数是："+ Arrays.asList(objects));
          System.out.println("方法开始执行，参数是：");
      }
  
      @AfterReturning("execution( public int com.mashibing.inter.MyCalculator.*(int,int))")
      public static void stop(){
  //        System.out.println("XXX方法执行结束，结果是："+ Arrays.asList(objects));
  //        System.out.println(method.getName()+"方法执行结束，结果是："+ Arrays.asList(objects));
          System.out.println("方法执行完成，结果是：");
  
      }
  
      @AfterThrowing("execution( public int com.mashibing.inter.MyCalculator.*(int,int))")
      public static void logException(){
  //        System.out.println(method.getName()+"方法出现异常："+ e.getMessage());
          System.out.println("方法出现异常：");
      }
  
      @After("execution( public int com.mashibing.inter.MyCalculator.*(int,int))")
      public static void end(){
  //        System.out.println(method.getName()+"方法执行结束了......");
          System.out.println("方法执行结束了......");
      }
  }
  ```

- 开启基于注解的aop的功

- <context:component-scan base-package="com.mashibing"></context:component-scan>

- xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         
  导入context       
         xmlns:context="http://www.springframework.org/schema/context"
  导入AOP       
         xmlns:aop="http://www.springframework.org/schema/aop"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans.xsd
         http://www.springframework.org/schema/context
         http://www.springframework.org/schema/context/spring-context.xsd
                             
         http://www.springframework.org/schema/aop
         https://www.springframework.org/schema/aop/spring-aop.xsd
  ">
      <context:component-scan base-package="com.mashibing"></context:component-scan>
      <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
  </beans>
  ```

### 3、测试

  MyTest.java

  ```java
  import com.mashibing.inter.Calculator;
  import org.springframework.context.ApplicationContext;
  import org.springframework.context.support.ClassPathXmlApplicationContext;
  public class MyTest {
      public static void main(String[] args){
          ApplicationContext context = new ClassPathXmlApplicationContext("aop.xml");
          Calculator bean = context.getBean(Calculator.class);
          bean.add(1,1);
      }
}
  ```

​		spring AOP的动态代理方式是jdk自带的方式，容器中保存的组件是代理对象com.sun.proxy.$Proxy对象

  ### 4、通过cglib来创建代理对象

MyCalculator.java

```java
package com.mashibing.inter;

import org.springframework.stereotype.Service;

@Service
public class MyCalculator {
    public int add(int i, int j) {
        int result = i + j;
        return result;
    }

    public int div(int i, int j) {
        int result = i / j;
        return result;
    }
}
```

MyTest.java

```java
public class MyTest {
    public static void main(String[] args){
        ApplicationContext context = new ClassPathXmlApplicationContext("aop.xml");
        MyCalculator bean = context.getBean(MyCalculator.class);
        bean.add(1,1);
        System.out.println(bean);
        System.out.println(bean.getClass());
    }
}
```

​		可以通过cglib的方式来创建代理对象，此时不需要实现任何接口，代理对象是

class com.mashibing.inter.MyCalculator$$EnhancerBySpringCGLIB$$1f93b605类型

​		**综上所述：在spring容器中，如果有接口，那么会使用jdk自带的动态代理，如果没有接口，那么会使用cglib的动态代理。动态代理的实现原理，后续会详细讲。**

### 5、表达式的抽取

如果在实际使用过程中，多个方法的表达式是一致的话，那么可以考虑将切入点表达式抽取出来：

​		a、随便声明一个没有实现的返回void的空方法

​		b、给方法上标注@Potintcut注解

```java
@Component
@Aspect
public class LogUtil {

    //镶嵌在Proxy中，可以用来生成日志
    @Pointcut("execution(public Integer com.dzp..*(Integer,Integer))")
    public void myPoint(){
    }

    
    @Before(value = "myPoint()")
    public void start(JoinPoint joinPoint){
        Object[] args = joinPoint.getArgs();
        Signature signature = joinPoint.getSignature();
        String name = signature.getName();
        System.out.println(name+"正在执行，输入的数值是"+Arrays.asList(args));
    }

    @After(value = "myPoint()")
    public void logFinal(JoinPoint joinPoint){
        String name = joinPoint.getSignature().getName();
        System.out.println(name+"结束执行");
    }

    @AfterReturning(value = "myPoint()",returning = "result")
    public void stop(JoinPoint joinPoint,Object result){
        String name = joinPoint.getSignature().getName();
        System.out.println(name+"输出完毕，结果是"+result);
    }


}
```

Test

```java
public class Test_2 {
    @Test
    public void test(){
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        MyCalculator calculator1 = context.getBean("myCalculator", MyCalculator.class);
        calculator1.add(2,3);
    }
}
```

### 		6、环绕通知的使用

LogUtil.java

```java
@Component
@Aspect
public class LogUtil {
    @Pointcut("execution( public int com.mashibing.inter.MyCalculator.*(int,int))")
    public void myPoint(){}
    
    /**
     * 环绕通知是spring中功能最强大的通知
     * @param proceedingJoinPoint
     * @return
     */
    @Around("myPoint()")
    public Object myAround(ProceedingJoinPoint proceedingJoinPoint){
        Object[] args = proceedingJoinPoint.getArgs();
        String name = proceedingJoinPoint.getSignature().getName();
        Object proceed = null;
        try {
            System.out.println("环绕前置通知:"+name+"方法开始，参数是"+Arrays.asList(args));
            //利用反射调用目标方法，就是method.invoke()
            proceed = proceedingJoinPoint.proceed(args);
            System.out.println("环绕返回通知:"+name+"方法返回，返回值是"+proceed);
        } catch (Throwable e) {
            System.out.println("环绕异常通知"+name+"方法出现异常，异常信息是："+e);
        }finally {
            System.out.println("环绕后置通知"+name+"方法结束");
        }
        return proceed;
    }
}
```

​		总结：环绕通知的执行顺序是优于普通通知的，具体的执行顺序如下：

环绕前置-->普通前置-->目标方法执行-->环绕正常结束/出现异常-->环绕后置-->普通后置-->普通返回或者异常。

但是需要注意的是，如果出现了异常，那么环绕通知会处理或者捕获异常，普通异常通知是接收不到的，因此最好的方式是在环绕异常通知中向外抛出异常。



### 7、多切面运行的顺序

在spring中，默认是按照切面名称的字典顺序进行执行的，但是如果想自己改变具体的执行顺序的话，可以使用@Order注解来解决，数值越小，优先级越高。



LogUtil.java

~~~java
@Component
@Aspect
@Order(2)
public class LogUtil {}
~~~

SecurityAspect.java

~~~java
@Component
@Aspect
@Order(1)
public class SecurityAspect {}
~~~



## 声明式事务

### 事务配置的属性

​		isolation：设置事务的隔离级别

​		propagation：事务的传播行为

​		noRollbackFor：那些异常事务可以不回滚

​		noRollbackForClassName：填写的参数是全类名

​		rollbackFor：哪些异常事务需要回滚

​		rollbackForClassName：填写的参数是全类名

​		readOnly：设置事务是否为只读事务		

​		timeout：事务超出指定执行时长后自动终止并回滚,单位是秒

### 事务的传播特性

事务的传播特性指的是当一个事务方法被另一个事务方法调用时，这个事务方法应该如何进行？

spring的事务传播行为一共有7种：

| 传播属性     | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| REQUIRED     | 如果此方法外层有事务在运行，那当前方法就归属于外层的事务，否则启动新的自己的事务 |
| REQUIRED_NEW | REQUIRED_NEW中报错时，会导致另一个REQUIRED回滚（外层捕捉到了异常，）<br />REQUIRED报错时，另一个REQUIRED_NEW不会回滚。 |

REQUIRED_NEW中报错时，会导致另一个REQUIRED回滚（外层捕捉到了异常，）

REQUIRED报错时，另一个REQUIRED_NEW不会回滚。

![传播特性](E:\deng\image\传播特性.png)







# 原理

# 06Spring原理讲解

### 1、什么是Spring框架，Spring框架主要包含哪些模块

​		Spring是一个开源框架，Spring是一个轻量级的Java 开发框架。它是为了解决企业应用开发的复杂性而创建的。框架的主要优势之一就是其分层架构，分层架构允许使用者选择使用哪一个组件，同时为 J2EE 应用程序开发提供集成的框架。Spring使用基本的JavaBean来完成以前只可能由EJB完成的事情。然而，Spring的用途不仅限于服务器端的开发。从简单性、可测试性和松耦合的角度而言，任何Java应用都可以从Spring中受益。Spring的核心是控制反转（IoC）和面向切面（AOP）。简单来说，Spring是一个分层的full-stack(一站式) 轻量级开源框架。

主要包含的模块：

![](E:/deng/deng/MD/spring/image/spring-overview.png)

![img](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1599047126824&di=39225eb78eddd211065f5c46f1d81b59&imgtype=0&src=http%3A%2F%2Fww4.sinaimg.cn%2Fmw690%2F7178f37egw1etwb5nakx1j20k00f0dhy.jpg)

### 2、Spring框架的优势

​		1、Spring通过DI、AOP和消除样板式代码来简化企业级Java开发

​		2、Spring框架之外还存在一个构建在核心框架之上的庞大生态圈，它将Spring扩展到不同的领域，如Web服务、REST、移动开发以及NoSQL

​		3、低侵入式设计，代码的污染极低

​		4、独立于各种应用服务器，基于Spring框架的应用，可以真正实现Write Once,Run Anywhere的承诺

​		5、Spring的IoC容器降低了业务对象替换的复杂性，提高了组件之间的解耦

​		6、Spring的AOP允许将一些通用任务如安全、事务、日志等进行集中式处理，从而提供了更好的复用

​		7、Spring的ORM和DAO提供了与第三方持久层框架的的良好整合，并简化了底层的数据库访问

​		8、Spring的高度开放性，并不强制应用完全依赖于Spring，开发者可自由选用Spring框架的部分或全部

### 3、IOC和DI是什么？

​		控制反转是就是应用本身不负责依赖对象的创建和维护,依赖对象的创建及维护是由外部容器负责的,这样控制权就有应用转移到了外部容器,控制权的转移就是控制反转。

​		依赖注入是指:在程序运行期间,由外部容器动态地将依赖对象注入到组件中如：一般，通过构造函数注入或者setter注入。

### 4、描述下Spring IOC容器的初始化过程

​		Spring IOC容器的初始化简单的可以分为三个过程：

​		第一个过程是Resource资源定位（xml）。这个Resouce指的是BeanDefinition的资源定位。这个过程就是容器找数据的过程，就像水桶装水需要先找到水一样。

​		第二个过程是BeanDefinition的载入过程。这个载入过程是把用户定义好的Bean表示成Ioc容器内部的数据结构，而这个容器内部的数据结构就是BeanDefition。

​		第三个过程是向IOC容器注册这些BeanDefinition的过程，这个过程就是将前面的BeanDefition保存到HashMap中的过程。

### 5、BeanFactory 和 FactoryBean的区别？

- **BeanFactory**是个Factory，也就是IOC容器或对象工厂，在Spring中，所有的Bean都是由BeanFactory(也就是IOC容器)来进行管理的，提供了实例化对象和拿对象的功能。

  使用场景：

  - 从Ioc容器中获取Bean(byName or byType)
  - 检索Ioc容器中是否包含指定的Bean
  - 判断Bean是否为单例

- **FactoryBean**是个Bean，这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean,它的实现与设计模式中的工厂模式和修饰器模式类似。

  使用场景

  - ProxyFactoryBean

### 6、BeanFactory和ApplicationContext的异同

![](E:/deng/deng/MD/spring/image/ApplicationContext类图.png)

相同：

- Spring提供了两种不同的IOC 容器，一个是BeanFactory，另外一个是ApplicationContext，它们都是Java  interface，ApplicationContext继承于BeanFactory(ApplicationContext继承ListableBeanFactory。
- 它们都可以用来配置XML属性，也支持属性的自动注入。
- 而ListableBeanFactory继承BeanFactory)，BeanFactory 和 ApplicationContext 都提供了一种方式，使用getBean("bean name")获取bean。

不同：

- 当你调用getBean()方法时，BeanFactory仅实例化bean，而ApplicationContext 在启动容器的时候实例化单例bean，不会等待调用getBean()方法时再实例化。
- BeanFactory不支持国际化，即i18n，但ApplicationContext提供了对它的支持。
- BeanFactory与ApplicationContext之间的另一个区别是能够将事件发布到注册为监听器的bean。
- BeanFactory 的一个核心实现是XMLBeanFactory 而ApplicationContext  的一个核心实现是ClassPathXmlApplicationContext，Web容器的环境我们使用WebApplicationContext并且增加了getServletContext 方法。
- 如果使用自动注入并使用BeanFactory，则需要使用API注册AutoWiredBeanPostProcessor，如果使用ApplicationContext，则可以使用XML进行配置。
- 简而言之，BeanFactory提供基本的IOC和DI功能，而ApplicationContext提供高级功能，BeanFactory可用于测试和非生产使用，但ApplicationContext是功能更丰富的容器实现，应该优于BeanFactory

### 7、Spring Bean 的生命周期？

![](E:/deng/deng/MD/spring/image/bean生命周期.png)

总结：

**（1）实例化Bean：**

对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean进行实例化。对于ApplicationContext容器，当容器启动结束后，通过获取BeanDefinition对象中的信息，实例化所有的bean。

**（2）设置对象属性（依赖注入）：**

实例化后的对象被封装在BeanWrapper对象中，紧接着，Spring根据BeanDefinition中的信息 以及 通过BeanWrapper提供的设置属性的接口完成依赖注入。

**（3）处理Aware接口：**

接着，Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给Bean：

①如果这个Bean已经实现了BeanNameAware接口，会调用它实现的setBeanName(String beanId)方法，此处传递的就是Spring配置文件中Bean的id值；

②如果这个Bean已经实现了BeanFactoryAware接口，会调用它实现的setBeanFactory()方法，传递的是Spring工厂自身。

③如果这个Bean已经实现了ApplicationContextAware接口，会调用setApplicationContext(ApplicationContext)方法，传入Spring上下文；

**（4）BeanPostProcessor：**

如果想对Bean进行一些自定义的处理，那么可以让Bean实现了BeanPostProcessor接口，那将会调用postProcessBeforeInitialization(Object obj, String s)方法。

**（5）InitializingBean 与 init-method：**

如果Bean在Spring配置文件中配置了 init-method 属性，则会自动调用其配置的初始化方法。

**（6）如果这个Bean实现了BeanPostProcessor接口**，将会调用postProcessAfterInitialization(Object obj, String s)方法；由于这个方法是在Bean初始化结束时调用的，所以可以被应用于内存或缓存技术；

以上几个步骤完成后，Bean就已经被正确创建了，之后就可以使用这个Bean了。

**（7）DisposableBean：**

当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean这个接口，会调用其实现的destroy()方法；

**（8）destroy-method：**

最后，如果这个Bean的Spring配置中配置了destroy-method属性，会自动调用其配置的销毁方法。

### 8、Spring AOP的实现原理？

​		Spring AOP使用的动态代理，所谓的动态代理就是说AOP框架不会去修改字节码，而是在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法。

​		Spring AOP中的动态代理主要有两种方式，JDK动态代理和CGLIB动态代理。JDK动态代理通过反射来接收被代理的类，并且要求被代理的类必须实现一个接口。JDK动态代理的核心是InvocationHandler接口和Proxy类。

​		如果目标类没有实现接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成某个类的子类，注意，CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。

### 9、Spring 是如何管理事务的？

​		Spring事务管理主要包括3个接口，Spring的事务主要是由它们(**PlatformTransactionManager，TransactionDefinition，TransactionStatus**)三个共同完成的。

**1. PlatformTransactionManager**：事务管理器--主要用于平台相关事务的管理

主要有三个方法：

- commit 事务提交；
- rollback 事务回滚；
- getTransaction 获取事务状态。

**2. TransactionDefinition**：事务定义信息--用来定义事务相关的属性，给事务管理器PlatformTransactionManager使用

这个接口有下面四个主要方法：

- getIsolationLevel：获取隔离级别；
- getPropagationBehavior：获取传播行为；
- getTimeout：获取超时时间；
- isReadOnly：是否只读（保存、更新、删除时属性变为false--可读写，查询时为true--只读）

事务管理器能够根据这个返回值进行优化，这些事务的配置信息，都可以通过配置文件进行配置。

**3. TransactionStatus**：事务具体运行状态--事务管理过程中，每个时间点事务的状态信息。

例如它的几个方法：

- hasSavepoint()：返回这个事务内部是否包含一个保存点，
- isCompleted()：返回该事务是否已完成，也就是说，是否已经提交或回滚
- isNewTransaction()：判断当前事务是否是一个新事务

**声明式事务的优缺点**：

- **优点**：不需要在业务逻辑代码中编写事务相关代码，只需要在配置文件配置或使用注解（@Transaction），这种方式没有侵入性。
- **缺点**：声明式事务的最细粒度作用于方法上，如果像代码块也有事务需求，只能变通下，将代码块变为方法。

### 10、Spring 的不同事务传播行为有哪些，干什么用的？

![](E:/deng/deng/MD/spring/image/传播特性.jpg)

### 11、Spring 中用到了那些设计模式？

- 代理模式—在AOP中被用的比较多。
- 单例模式—在spring配置文件中定义的bean默认为单例模式。
- 模板方法—用来解决代码重复的问题。比如. RestTemplate, JmsTemplate, JpaTemplate。
- 工厂模式—BeanFactory用来创建对象的实例。
- 适配器--spring aop
- 装饰器--spring data hashmapper
- 观察者-- spring 事件驱动模型
- 回调--Spring Aware回调接口

### 12、Spring如何解决循环依赖？

https://blog.csdn.net/qq_36381855/article/details/79752689

### 13、bean的作用域

（1）singleton：默认，每个容器中只有一个bean的实例，单例的模式由BeanFactory自身来维护。

（2）prototype：为每一个bean请求提供一个实例。

（3）request：为每一个网络请求创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收。

（4）session：与request范围类似，确保每个session中有一个bean的实例，在session过期后，bean会随之失效。

### 14、Spring框架中有哪些不同类型的事件

（1）上下文更新事件（ContextRefreshedEvent）：在调用ConfigurableApplicationContext 接口中的refresh()方法时被触发。

（2）上下文开始事件（ContextStartedEvent）：当容器调用ConfigurableApplicationContext的Start()方法开始/重新开始容器时触发该事件。

（3）上下文停止事件（ContextStoppedEvent）：当容器调用ConfigurableApplicationContext的Stop()方法停止容器时触发该事件。

（4）上下文关闭事件（ContextClosedEvent）：当ApplicationContext被关闭时触发该事件。容器被关闭时，其管理的所有单例Bean都被销毁。

（5）请求处理事件（RequestHandledEvent）：在Web应用中，当一个http请求（request）结束触发该事件。

### 15、Spring通知有哪些类型

（1）前置通知（Before advice）：在某连接点（join point）之前执行的通知，但这个通知不能阻止连接点前的执行（除非它抛出一个异常）。

（2）返回后通知（After returning advice）：在某连接点（join point）正常完成后执行的通知：例如，一个方法没有抛出任何异常，正常返回。 

（3）抛出异常后通知（After throwing advice）：在方法抛出异常退出时执行的通知。 

（4）后通知（After (finally) advice）：当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。 

（5）环绕通知（Around Advice）：包围一个连接点（join point）的通知，如方法调用。这是最强大的一种通知类型。 环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它们自己的返回值或抛出异常来结束执行。 环绕通知是最常用的一种通知类型。

### 16、Spring的自动装配

在spring中，对象无需自己查找或创建与其关联的其他对象，由容器负责把需要相互协作的对象引用赋予各个对象，使用autowire来配置自动装载模式。

在Spring框架xml配置中共有5种自动装配：

（1）no：默认的方式是不进行自动装配的，通过手工设置ref属性来进行装配bean。

（2）byName：通过bean的名称进行自动装配，如果一个bean的 property 与另一bean 的name 相同，就进行自动装配。 

（3）byType：通过参数的数据类型进行自动装配。

（4）constructor：利用构造函数进行装配，并且构造函数的参数通过byType进行装配。

（5）autodetect：自动探测，如果有构造方法，通过 construct的方式自动装配，否则使用 byType的方式自动装配。

基于注解的方式：

使用@Autowired注解来自动装配指定的bean。在使用@Autowired注解之前需要在Spring配置文件进行配置，<context:annotation-config />。在启动spring IoC时，容器自动装载了一个AutowiredAnnotationBeanPostProcessor后置处理器，当容器扫描到@Autowied、@Resource或@Inject时，就会在IoC容器自动查找需要的bean，并装配给该对象的属性。在使用@Autowired时，首先在容器中查询对应类型的bean：

如果查询结果刚好为一个，就将该bean装配给@Autowired指定的数据；

如果查询的结果不止一个，那么@Autowired会根据名称来查找；

如果上述查找的结果为空，那么会抛出异常。解决方法时，使用required=false。

@Autowired可用于：构造函数、成员变量、Setter方法

注：@Autowired和@Resource之间的区别

(1) @Autowired默认是按照类型装配注入的，默认情况下它要求依赖对象必须存在（可以设置它required属性为false）。

(2) @Resource默认是按照名称来装配注入的，只有当找不到与名称匹配的bean才会按照类型来装配注入。



# 循环依赖

全为单例时可以

prototype会检查引用，如果有引用对象没有实例化，暂停自己的实例化，转而去实例化引用对象