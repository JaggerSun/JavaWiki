Spring 用Resource管理资源,提供exists, getURL, getFile等等接口。
用来处理Spring中的各种类型的文件比如ClassPath下面的，URL中的等等，
其包括了
UrlResouce
ClassPathResource
FileSystemResource等等。
使用 http:
file:等
Resource template = ctx.getResource("classpath:some/resource/path/myTemplate.txt")


数据校验、绑定，PropertyEditor
校验是否当做业务逻辑，是否要绑定在Web层，应该很容易本地化并且可以翻遍的加入新的验证逻辑。  Spring提供Validator接口，可以适用于应用程序的任何一个层面。
DataBinding  把用户输入和domain对象绑定起来。  与Validator结合，主要拥有MVC框架
PropertyEditor是JavaBean规范的一部分
BeanWrapper company = BeanWrapperImpl(new Company());
// setting the company name..
company.setPropertyValue("name", "Some Company Inc.");

BeanWrapper是包装类， 也是规范的一部分，用来操作bean的属性
PropertyEditor是String到Object的转换规则

这一部分在Spring MVC部分我再具体的研究



Spring Expression Language
数学表达式
<bean id="numberGuess" class="org.spring.samples.NumberGuess">
    <property name="randomNumber" value="#{ T(java.lang.Math).random() * 100.0 }"/>

    <!-- other properties -->
</bean>

来源： <http://docs.spring.io/spring/docs/3.2.13.RELEASE/spring-framework-reference/htmlsingle/#beans>
 
系统属性
<bean id="taxCalculator" class="org.spring.samples.TaxCalculator">
    <property name="defaultLocale" value="#{ systemProperties['user.region'] }"/>

    <!-- other properties -->
</bean>
bean属性
<bean id="shapeGuess" class="org.spring.samples.ShapeGuess">
    <property name="initialShapeSeed" value="#{ numberGuess.randomNumber }"/>

    <!-- other properties -->
</bean>
还是有不少，慢慢再说。不慌，可以用来作为备用的代码库使用




