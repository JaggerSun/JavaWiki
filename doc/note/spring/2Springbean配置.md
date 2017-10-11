Bean的容器  Factory和Context,Context多了运行时上下文，提供读取资源文件，监听等等能力

Bean的生命周期：
实例化->填充属性->BeanNameAware的setBeanName方法， 调用BeanFactoryAware的setBeanFactory()方法，调用ApplicationContextAware的setApplicationContext()方法——》调用BeanPostProcessore的预初始化方法，调用InitializingBean的afterPropertiesSet()方法，调用定制的初始化方法用init-method声明的，调用BeanPostProcessors的后初始化方法。

容器关闭->调用DisposableBeande destroy()方法，调用定制的销毁方法-用destory-method声明的

全注解的方式：
@Configuration
public class AppConfig {
    private @Value("#{jdbcProperties.url}") String jdbcUrl;
    private @Value("#{jdbcProperties.username}") String username;
    private @Value("#{jdbcProperties.password}") String password;

    @Bean
    public FooService fooService() {
        return new FooServiceImpl(fooRepository());
    }

    @Bean
    public FooRepository fooRepository() {
        return new HibernateFooRepository(sessionFactory());
    }

    @Bean
    public SessionFactory sessionFactory() {
        // wire up a session factory
        AnnotationSessionFactoryBean asFactoryBean =
            new AnnotationSessionFactoryBean();
        asFactoryBean.setDataSource(dataSource());
        // additional config
        return asFactoryBean.getObject();
    }

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource(jdbcUrl, username, password);
    }
}

来源： <http://docs.spring.io/spring/docs/3.2.13.RELEASE/spring-framework-reference/htmlsingle/>
 

Spring IOC
org.springframework.beans和.context下完成的，beans的BeanFactory提供了IOC容器， context提供了资源管理，事件发布等

基本：
1. setter注入
    <bean><property name="" value|ref=""></bean>
2. 构造器注入
     <bean><constructor-arg value|ref=""></bean>
3. 内部bean:
        <property name=""><bean class=""/></>
4. 还可以用<bean p:propertyName="">这样的方式注入
5. 装配集合，properties文件样式的键值对
4. 空值<null/>
5. 自动装配 autowire="byName"，命中多个会报错。还有其他，掠过
6. <bean scope=
    默认singleton, prototype每次调用一个新的，request,session，这两个作用于仅仅在Spring MVC中才有用范围 scope=  sngleton默认值，容器一个，  Prototype使用一次创建一个
7. 利用工厂方法创建bean 
factory-method="methodName" methodName是一个静态方法返回该类的实例，特别适合用Spring管理单例
这样的感觉不太有用。  因为返回的是直接new的值， 不带有Spring的环境。 感觉可以使用SpringHolder来实现参数选择不同实现的那种工厂。没有想到更好的办法。

8. 初始化和析构
    <bean init-method="methodName"
    配置的办法init-method和destroy-method.       编程的办法实现这两个接口：InitializingBean , DisposableBean atfter和destory
    不同在于编程式的不需要额外的配置，Spring容器会自动处理。配置式的可以跟Spring不耦合。  
    都是在实例创建之后调用的  
@PostConstruct and @PreDestroy

SEL spring EL, 跟JSP中的EL差不多。  用#{beanId},#{properties}等等这样的方式，还能使用正则表达式等等

注解装配：
<context:annotation-config>
@Autowired 可以用在set方法，构造器，通常是用在属性上，然后省略set和get,其为byType装配，但是如果没有或者有多个都会报错：
没有@Autowired(request=false)会填空值
多个加上@Qualifier("id")来缩小范围
@Resource基于byName的，是JSR的规范，更小的依赖于Spring
@Inject JSR330,专门制定的用于依赖的注解，可以完全去掉Autowired

自动扫描bean,上面的实现了去除<properties及<constract但是仍然需要<bean,
下面来消除bean:
<context:component-scan base-package
@Component
@Controller
@Repository
@Service
上面这几个现在没有区别，以后可能会有，因此最好还是按照分层思想来定义

========================


高级bean装配
1. 继承   被继承的abstract="true"(表示spring不负责实例化只是作为模板),子类parent="parentId"
2. 方法注入   <replace-method name="methodName" replacer="beanId"><bean id=""> 这个bean需要实现了spring的MethodReplacer类
3. 自定义属性编辑器 PropertyEidtor和PropertyEidtorSupport类
    <bean class="CustomEditorConfigurer"><property name="customEditors"><map><entry key="要解析的类的名称"><bean 使用的解析类的名称>
4. 后处理bean BeanPostProcessor,  postProcessBeforeInitialization.在调用bean的init方法之前实例化之后，postAfter,在调用init方法之后。ApplicationContextAwareProcessor
5.  BeanFactoryPostProcessor在spring加载且任何bean被实例化之前CustomerEditorConfigurer,PropertyPlaceholderConfigurer
6. 用属性文件需要AppliationContext,   PropertyPlaceholderConfigurer指定属性文件的位置，其他配置用EL表达式使用。
7. 国际化  
    国际化文件
    在bean里装配ResourceBundleMessageSource(名称必须为messageSource)来使用java自己的ResourceBundle来提取消息,会被自动注入到context中用context.getMessage来获取信息，在JSP中可以用<spring:message code="">来获取信息
8. 事件解耦
    发布事件:context.publishEvent(new 实现ApplicationEvent)
    监听事件:实现ApplicationListener并且声明为bean。当事件发布的时候会调用它的onApplicationEvent方法。有点想servlet的filter,比如调用publishEvent的这个时间点就会一次调用所有已经声明的Listener，需要注意的是这个是同步的，因此要注意效率
9. 跟容器关联
    BeanNameAwae, ApplicationContextAware。实现之后spring会自动注入。或者使用ContextHolder.
10. 脚本化
    <property name="" ref="">这个部分跟正常的完全一样，然后ref部分我们使用一个脚本语言：
    <lang:groovy id="ref" script-source="classpath:com/....groovy"/>默认是不刷新的，可以定义刷新时间和使用内部代码的方式。

 
    