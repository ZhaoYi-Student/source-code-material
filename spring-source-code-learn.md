### spring source code learn
### @auth : chen

#### Spring是什么
- spring 是一个框架，是一个生态，为了简化开发，解耦为目的产生的

#### Spring结构
+ IOC
  - IOC控制反转
    * 解析指定xml文件，获取到对应的bean，放到一个容器中进行统一的生态管理，这个容器就是IOC容器。与传统使用对象相比，IOC自动帮我们管理的对象的创建，使用，销毁
  - DI依赖注入
    * 如下，在运行spring项目的时候，会有过程，根据注解或者构造函数的方式获取到beanfactory中的bean实力对象，也就是从IOC容器中获取，并付给类中的属性。
+ AOP
  - AOP是一种切面的概念思想，可以将公共的，且与业务逻辑无关的代码抽取出来进行统一的管理。
  - 其原理就是通过代理思想，在spring启动的时候根据切点表达式判断该bean是否符合条件，如果符合将这个bean实力变成一个代理对象，进行前置，后置，或者环绕通知

1. spring 使用
首先创建一个xxx.xml文件,如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:msb="http://www.mashibing.com/schema/user"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context.xsd
        http://www.mashibing.com/schema/user http://www.mashibing.com/schema/user.xsd">

    <msb:user id="msb" userName="lian" email="lian@msb.com" password="123456"></msb:user>
    <context:component-scan base-package="com.mashibing" ></context:component-scan>
    <bean id="user" class="com.mashibing.supplier.User" ></bean>
    <context:property-placeholder location="classpath:db.properties" ></context:property-placeholder>
    <bean class="com.mashibing.selfbdrpp.MyBeanDefinitionRegistryPostProcessor"></bean>
    <bean id="person2" class="com.mashibing.Person" depends-on="">
        <constructor-arg index="0" value="2">
        </constructor-arg>
        <constructor-arg index="1" value="lisi"></constructor-arg>
    </bean>
    <bean id="person"  class="com.mashibing.Person" scope="prototype">
        <property name="id" value="1"></property>
        <property name="name" value="zhangsan"></property>
    </bean>
    <bean name="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
            <property name="username" value="${jdbc.username}"></property>
    </bean>

    <bean id="studentConverter" class="com.mashibing.selfConverter.StudentConverter"></bean>
    <bean id ="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <ref bean="studentConverter"></ref>
            </set>
        </property>
    </bean>
    <bean class="com.mashibing.MyBeanFactoryPostProcessor"></bean>
    <bean id="a" class="com.mashibing.cycle.A" scope="prototype">
        <property name="b" ref="b"></property>
    </bean>
    <bean id="b" class="com.mashibing.cycle.B" scope="prototype">
        <property name="a" ref="a"></property>
    </bean>
</beans>
```

#### Spring 启动过程
1. 加载xml文件
2. 解析xml文件
3. 封装beandefinition对象
4. 实例化并放到容器中
5. 属性赋值
6. 初始化
7. 从容器中获取


#### Spring mvc 九大内置对象
1. HandlerMapper
2. HandlerAdapter
3. HandlerExceptionResolver
4. ViewResolver
5. RequestToViewNameTranslator
6. LocalResolver
7. ThemeResolver
8. MultipartResolver
9. FlashMapManager

#### Spring mvc 请求过程  
1. Spring mvc实现了HttpServlet接口，所以请求会先调用service方法，而Spring mvc使用FrameworkServlet实现了HttpServlet接口，所以先调用了service，在service获取请求方式，根据请求方式调用相关方法（get，post，put，delete等）
2. 每个请求方法都会调用统一的方法processRequest，获取一些属性后调用方法service设置一些属性之后调用核心方法doDispatch
3. 判断是否上传请求，如果是上传请求要特殊处理
4. 根据request请求获取handler对象
5. 根据handler找到handlerAdapter
6. 处理laster modified，如果资源没有发生变化，读取缓存
7. 执行intercept的preHandler方法
8. 判断是否需要异步处理，是的话直接返回
9. 当view为空时，设置默认的view对象
10. 执行intercept的postHandler方法
11. 如果发生异常情况，将异常设置到dispatcher中，交给HandlerExceptionResolver处理
12. 设置view，进行页面的渲染操作
13. 如果发生异常，执行interceptor的afterCompletion方法
