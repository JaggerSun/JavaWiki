http://projects.spring.io/spring-framework/
点这只猫
https://github.com/spring-projects/spring-framework/wiki/Downloading-Spring-artifacts
http://repo.spring.io/webapp/search/artifact/?18&q=spring

Spring源码下载地址：
直接使用github的svn下载功能：
https://github.com/spring-projects/spring-framework.git/tags/v3.0.5.RELEASE/org.springframework.jms
https://github.com/spring-projects/spring-framework/branches/3.0.x/org.springframework.aspects

简化企业应用开发的复杂性。
使用IOC和AOP来统一了应用对象的查找、配置、和生命周期管理，分离了业务和基础服务中的不同关注点。开发人员可以基于简单Java对象轻松地实现与EJB同样强大的功能。

在业务层提供了全面的解决方案，包括：数据库持久化支持、声明式事务、远程服务访问，以及JMS，Mail,定时等多种企业服务。

在Web层提供了MVC框架，并且可以集成各种Web框架或者试图技术(JSP,Velocity)

IOC帮我们省略了大量的定制工厂和系统配置类
Hibernate帮我们省略了大量的对象/关系映射，数据库连接代码。 二者整合有省去了大量的对于操作Hibernate所需要的代码

Spring的设计理念： 好的设计比实现技术更重要，模型上接口松散耦合，代码应该容易被测试
Spring核心特性：
        IOC，  不是类去找其依赖的对象，而是容器把依赖注入给他。
        AOP,    两个图很好的说明了， 一个是多线图，一个是分层图
        模板， 最典型的是数据库连接模板

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/spring/spring-1.jpg)

核心容器：
负责Bean创建和配置IOC-core and bean
Context 在Core-bean基础上提供一些框架的支持，比如国际化，资源加载，JMX, 事件传播等
EL  是spring的扩展语言可以不依赖于Spring容器直接调用，ExpressionParser parser = new SpelExpressionParser();
// invokes 'getBytes().length'
Expression exp = parser.parseExpression("'Hello World'.bytes.length");
<bean class="mycompany.RewardsTestDatabase">
    <property name="databaseName"
        value="#{systemProperties.databaseName}"/>
    <property name="keyGenerator"
        value="#{strategyBean.databaseKeyGenerator}"/>
</bean>

来源： <http://docs.spring.io/spring/docs/3.2.13.RELEASE/spring-framework-reference/htmlsingle/>
 】

DA/I
JDBC   解决了JDBC大量样板代码以及不友好的JDBC错误码
ORM   集成了流行的关系映射API：JPA,Hbernate, iBatis等，spring可以让这些框架跟spring其他的特性比如声明式事务管理特性相结合。
OXM   提供了XML/对象的映射模板， 简化JAXB。
JMS    异步消息的支持
通过AOP支持事务

WEB
提供web功能集成：文件上传，面向web的Context
对于Servlet,提供MVC
对于Structs提供整合

AOP
面向切片，与aspectJ整合

Spring系统的其他部分：
Spring Web Flow: 基于流程的会话式Web应用(购物车)，
Spring Web Service
Spring Security
Spring Integration: 应用集成模式
Spring Batch 批处理
Spring DM

版本更新：
2.5 新特性：
对注解的支持
内嵌AspectJ
SqlJdbcTemplate使用命名参数

3.0 新特性
Spring MVC支持Rest
表达式语言
更多的注解支持
====================

spring asm在core.java中有实现，跟spring-asm中的会冲突。
spring 的DI和IOC。  spring分为了哪几个部分。

spring的schmel文件是放在beans.jar的xml.factory中的，默认回去classpath中找，如果没有会报错
BeanDefinitionDocumentReader.class.cast
代码中好处的学习。
每个方法入口处都Assert类做参数检查，如果有问题抛出IllegalArgumentException
遍历的时候都加入了Node node = nl.item(i);node instanceof Element
得到

spring中的设计模式
修饰器

spring流程
BeanFactory
构建
Resource  可以打开流的一个资源，主要是为了构建包含Spring配置的信息。

BeanFactory   BeanDefinitionReader两种(PropertiesBeanDefinitionReader, XmlBeanDefinitionReader);
XmlBeanFactory构造的行为
reader.loadBeanDefinitions(resouce)
        1. 把resource读取成Document.    ProReader读取为Properties.
        2. registerBeanDefinitions  用来处理Document和Properties的
        3. 

BeanFactory和ApplicationContext  向监听器发消息，加入了国际化的支持等
      配合，装在，分发bean.
      用构造器，setter, 内部class注入。
      自动装配，按照名字和类型等 autowire
       bean scope: singleton, request,prototype等scope=
       用工厂方法
       类的初始化和销毁。  <bean init-method="" destroy-method=""或者是本身这个类实现spring的两个接口：InitializingBean,DisposableBan.
高级类装载
       继承 parent,将会继承所有的<bean中的属性和bean下面的<property>
       