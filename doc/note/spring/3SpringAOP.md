像日志，事务，缓存，安全这样的业务，实际上很重要，充斥着系统的很多模块，但是却不是业务流程想真正关注的，AOP为我们除了委托，继承以外的第三个选择。


AOP术语：

顺序执行的时候需要在某一个点插入一些新的内容

切面：插入的这个内容就是切面，描述了在何时何处来完成新的内容。其包含了通知(做什么和何时做)和切点(何处)
连接点：能够插入内容的点，比如方法调用，抛出异常，修改字段时。 Spring AOP只支持方法的， Aspect支持修改字段等
切点：是符合一定规则的连接点。 在代码中使用一些通配符或者正则表达式来限定的。
通知：切面在何时要完成什么工作，Spring的分为了5种类型：before(),after(),after-returning(执行成功之后),after-throwing(执行异常之后),around(前后)
引入：把新的方法引入现有类。
织入：在目标加入切面创建代理的过程
      有编译时(通过特殊的编译器来修改代码Aspect)，
      类加载时  需要特殊的ClassLoader
      运行时   JDK的动态代理。 Spring AOP的实现。
能切的地方叫连接点

Spring在加载bean的时候创建的是老对象，运行的时候才会创建代理对象，因此不需要编译器来织入Spring AOP切面

Spring AOP 只支持方法连接点， 如果是接口使用Proxy类，类的话使用cglib生成子类。
     这个是传统的做法，实在是太麻烦了：知道就好
     1. 定义通知。  实现MethodBeforeAdvice等接口定义他的bean
     2. 定义切点。  用正则表达式<bean class=JdkRegexpMethodPointcut><property name="pattern".这个是JDK1.5的定义方式，如果想强制使用aspectJ可以使用AspectJExpressionPointcutAdvisor
     3. 把通知和切点组装为切面，用DefaultPointcutAdvisor这个通知者类，使用其他的可以简化第二步
     4. 使用ProxyFactoryBean 构建代理对象。  <bean class="ProxyFactoryBean" ><target><inerceptorname><proxyInterfaces>
          proxyInterfaces是target目标实现的接口，来告诉代理对象需要实现哪些接口
     按照4中的结果需要给所有需要做切片的对象添加ProxyFactoryBean. 可以稍稍简化为使用继承bean
     5. 创建自动代理。<bean class="DefaultAdvisorAutoProxyCreator">被识别为一个BeanPostProcessor。这样就可以省去所有的ProxyFactoryBean.
下面是现在的做法：
    定义被切的类： <bean:
    定义切点： 使用Aspect的表达式风格，只支持部分。
                execution(* com.prince.GetProduct.get(..))
                execution是唯一的执行匹配， 其他都是约束的，用来约束的有within, this，arg等等
    定义通知，随便搞个类就行，支持POJO
    用配置是切点和通知结合起来：
            <bean id="getProduct" class="com.prince.GetProduct"></bean>
     <bean id="auth" class="com.prince.Auth"></bean>
     <bean id="log" class="com.prince.Log"></bean>
     <aop:config>
         <aop:pointcut expression="execution(* com.prince.GetProduct.get(..))" id="getPc"/>
         <aop:aspect ref="auth">
             <aop:before pointcut-ref="getPc" method="auth"/>
         </aop:aspect>
         <aop:aspect ref="log">
             <aop:after pointcut-ref="getPc" method="success"/>
         </aop:aspect>
     </aop:config>
            除了之中以外，还有个特别适合记录执行时间的通知，环绕通知
            
            还有就是拦截参数了

            再有呢就是通过切面引入新的功能
            


标注似得做法。
     1. @AspectJ表示这个类是个切面
     2. @Pointcut 标注在方法(这个方法跟被切的并不相关只是为了加上注解)上，用来表示切点。符合什么格式
     3. @Before等来标注方法，表示通知
     4. 加入动态代理<aop:aspectj-autoproxy>。会被创建为一个AnnotationAwareAspectJAutoProxyCreateor,然后实现自动代理。

POJO切面
     使用<aop:advisor>等标签来使得POJO对象成为切片


下面是另外一个重头戏了：
注入AspectJ切面
    运用aspect的语法创建一个切面，主要问题在于用spring管理这个bean的时候，因为aspect的实例都由aspect运行时创建，因此用spring声明的时候不能用默认的构造器创建方式了，只能使用factory-method的方法，这个是因为aspect的对象有一个静态方法aspectOf,通过这个静态方法能够返回一个实例
     
