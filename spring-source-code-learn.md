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

#### Spring mvc 启动流程
1. 启动tomcat执行web.xml文件
2. 执行ContextLoaderListener中的contextInitialized方法进行spring容器的初始化
3. 判断root是否为空，不为空抛出异常
4. 判断context对象是否为空，为空创建默认的XmlWebApplicationContext
5. 调用configureAndRefreshWebApplicationContext方法，判断id，加载spring相关配置文件，创建spring容器
6. 将创建好的spring容器赋值给root对象
7. 创建好DispatcherServlet对象根据servlet生命周期首先调用init的方法初始化springmvc，配置好九大内置对象
8. 此时spring容器已经创建完成

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


#### Springboot 自动装配原理

- 核心注解 @SpringBootApplication 是一个组合注解，其中包括三个注解 @SpringBootConfiguration, @EnableAutoConfiguration, @ComponmentScan

```
/*
 * Copyright 2012-2020 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.springframework.boot.autoconfigure;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.springframework.beans.factory.support.BeanNameGenerator;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.boot.context.TypeExcludeFilter;
import org.springframework.context.annotation.AnnotationBeanNameGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import org.springframework.core.annotation.AliasFor;
import org.springframework.data.repository.Repository;

/**
 * Indicates a {@link Configuration configuration} class that declares one or more
 * {@link Bean @Bean} methods and also triggers {@link EnableAutoConfiguration
 * auto-configuration} and {@link ComponentScan component scanning}. This is a convenience
 * annotation that is equivalent to declaring {@code @Configuration},
 * {@code @EnableAutoConfiguration} and {@code @ComponentScan}.
 *
 * @author Phillip Webb
 * @author Stephane Nicoll
 * @author Andy Wilkinson
 * @since 1.2.0
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {

	/**
	 * Exclude specific auto-configuration classes such that they will never be applied.
	 * @return the classes to exclude
	 */
	@AliasFor(annotation = EnableAutoConfiguration.class)
	Class<?>[] exclude() default {};

	/**
	 * Exclude specific auto-configuration class names such that they will never be
	 * applied.
	 * @return the class names to exclude
	 * @since 1.3.0
	 */
	@AliasFor(annotation = EnableAutoConfiguration.class)
	String[] excludeName() default {};

	/**
	 * Base packages to scan for annotated components. Use {@link #scanBasePackageClasses}
	 * for a type-safe alternative to String-based package names.
	 * <p>
	 * <strong>Note:</strong> this setting is an alias for
	 * {@link ComponentScan @ComponentScan} only. It has no effect on {@code @Entity}
	 * scanning or Spring Data {@link Repository} scanning. For those you should add
	 * {@link org.springframework.boot.autoconfigure.domain.EntityScan @EntityScan} and
	 * {@code @Enable...Repositories} annotations.
	 * @return base packages to scan
	 * @since 1.3.0
	 */
	@AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
	String[] scanBasePackages() default {};

	/**
	 * Type-safe alternative to {@link #scanBasePackages} for specifying the packages to
	 * scan for annotated components. The package of each class specified will be scanned.
	 * <p>
	 * Consider creating a special no-op marker class or interface in each package that
	 * serves no purpose other than being referenced by this attribute.
	 * <p>
	 * <strong>Note:</strong> this setting is an alias for
	 * {@link ComponentScan @ComponentScan} only. It has no effect on {@code @Entity}
	 * scanning or Spring Data {@link Repository} scanning. For those you should add
	 * {@link org.springframework.boot.autoconfigure.domain.EntityScan @EntityScan} and
	 * {@code @Enable...Repositories} annotations.
	 * @return base packages to scan
	 * @since 1.3.0
	 */
	@AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
	Class<?>[] scanBasePackageClasses() default {};

	/**
	 * The {@link BeanNameGenerator} class to be used for naming detected components
	 * within the Spring container.
	 * <p>
	 * The default value of the {@link BeanNameGenerator} interface itself indicates that
	 * the scanner used to process this {@code @SpringBootApplication} annotation should
	 * use its inherited bean name generator, e.g. the default
	 * {@link AnnotationBeanNameGenerator} or any custom instance supplied to the
	 * application context at bootstrap time.
	 * @return {@link BeanNameGenerator} to use
	 * @see SpringApplication#setBeanNameGenerator(BeanNameGenerator)
	 * @since 2.3.0
	 */
	@AliasFor(annotation = ComponentScan.class, attribute = "nameGenerator")
	Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;

	/**
	 * Specify whether {@link Bean @Bean} methods should get proxied in order to enforce
	 * bean lifecycle behavior, e.g. to return shared singleton bean instances even in
	 * case of direct {@code @Bean} method calls in user code. This feature requires
	 * method interception, implemented through a runtime-generated CGLIB subclass which
	 * comes with limitations such as the configuration class and its methods not being
	 * allowed to declare {@code final}.
	 * <p>
	 * The default is {@code true}, allowing for 'inter-bean references' within the
	 * configuration class as well as for external calls to this configuration's
	 * {@code @Bean} methods, e.g. from another configuration class. If this is not needed
	 * since each of this particular configuration's {@code @Bean} methods is
	 * self-contained and designed as a plain factory method for container use, switch
	 * this flag to {@code false} in order to avoid CGLIB subclass processing.
	 * <p>
	 * Turning off bean method interception effectively processes {@code @Bean} methods
	 * individually like when declared on non-{@code @Configuration} classes, a.k.a.
	 * "@Bean Lite Mode" (see {@link Bean @Bean's javadoc}). It is therefore behaviorally
	 * equivalent to removing the {@code @Configuration} stereotype.
	 * @since 2.2
	 * @return whether to proxy {@code @Bean} methods
	 */
	@AliasFor(annotation = Configuration.class)
	boolean proxyBeanMethods() default true;

}

```

#### 三个注解的作用
1. @SpringBootConfiguration 表明该类是一个配置类，可以在该类中注册bean
2. @EnableAutoConfiguration 表明自动化配置，将一些配置文件自动装配到spring中
3. @ComponentScan 表明扫描包，将扫描到的bean注册到spring中

#### @EnableAutoConfiguration 注解讲解

```
/*
 * Copyright 2012-2022 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.springframework.boot.autoconfigure;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.springframework.boot.autoconfigure.condition.ConditionalOnBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.context.annotation.ImportCandidates;
import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
import org.springframework.boot.web.servlet.server.ServletWebServerFactory;
import org.springframework.context.annotation.Conditional;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.core.io.support.SpringFactoriesLoader;

/**
 * Enable auto-configuration of the Spring Application Context, attempting to guess and
 * configure beans that you are likely to need. Auto-configuration classes are usually
 * applied based on your classpath and what beans you have defined. For example, if you
 * have {@code tomcat-embedded.jar} on your classpath you are likely to want a
 * {@link TomcatServletWebServerFactory} (unless you have defined your own
 * {@link ServletWebServerFactory} bean).
 * <p>
 * When using {@link SpringBootApplication @SpringBootApplication}, the auto-configuration
 * of the context is automatically enabled and adding this annotation has therefore no
 * additional effect.
 * <p>
 * Auto-configuration tries to be as intelligent as possible and will back-away as you
 * define more of your own configuration. You can always manually {@link #exclude()} any
 * configuration that you never want to apply (use {@link #excludeName()} if you don't
 * have access to them). You can also exclude them via the
 * {@code spring.autoconfigure.exclude} property. Auto-configuration is always applied
 * after user-defined beans have been registered.
 * <p>
 * The package of the class that is annotated with {@code @EnableAutoConfiguration},
 * usually via {@code @SpringBootApplication}, has specific significance and is often used
 * as a 'default'. For example, it will be used when scanning for {@code @Entity} classes.
 * It is generally recommended that you place {@code @EnableAutoConfiguration} (if you're
 * not using {@code @SpringBootApplication}) in a root package so that all sub-packages
 * and classes can be searched.
 * <p>
 * Auto-configuration classes are regular Spring {@link Configuration @Configuration}
 * beans. They are located using {@link ImportCandidates} and the
 * {@link SpringFactoriesLoader} mechanism (keyed against this class). Generally
 * auto-configuration beans are {@link Conditional @Conditional} beans (most often using
 * {@link ConditionalOnClass @ConditionalOnClass} and
 * {@link ConditionalOnMissingBean @ConditionalOnMissingBean} annotations).
 *
 * @author Phillip Webb
 * @author Stephane Nicoll
 * @since 1.0.0
 * @see ConditionalOnBean
 * @see ConditionalOnMissingBean
 * @see ConditionalOnClass
 * @see AutoConfigureAfter
 * @see SpringBootApplication
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

	/**
	 * Environment property that can be used to override when auto-configuration is
	 * enabled.
	 */
	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

	/**
	 * Exclude specific auto-configuration classes such that they will never be applied.
	 * @return the classes to exclude
	 */
	Class<?>[] exclude() default {};

	/**
	 * Exclude specific auto-configuration class names such that they will never be
	 * applied.
	 * @return the class names to exclude
	 * @since 1.3.0
	 */
	String[] excludeName() default {};

}


```

@EnableAutoConfiguration中包括一个@Import(AutoConfigurationImportSelector.class)
其中@Import表明一第三方的方式引入
AutoConfigurationImportSelector 该类实现了 ImportSelector接口 并重写了selectImports方法，表明会将selectImports方法中返回的所有类路径的类自动装配到spring中

```

@Override
public String[] selectImports(AnnotationMetadata annotationMetadata) {
  if (!isEnabled(annotationMetadata)) {
    return NO_IMPORTS;
  }
  // 获取所有自动化装配的bean
  AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
  return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
}

protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
  if (!isEnabled(annotationMetadata)) {
    return EMPTY_ENTRY;
  }
  // 获取所有EnableAutoConfiguration注解修饰的类放到map中
  AnnotationAttributes attributes = getAttributes(annotationMetadata);
  // 获取所有EnableAutoConfiguration 和 AutoConfiguration 注解修饰的bean
  List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
  // 移除相同属性
  configurations = removeDuplicates(configurations);
  // 获取排除项
  Set<String> exclusions = getExclusions(annotationMetadata, attributes);
  // 校验排除项
  checkExcludedClasses(configurations, exclusions);
  // 移除所有排除项的配置
  configurations.removeAll(exclusions);
  // 获取过滤项并过滤掉
  configurations = getConfigurationClassFilter().filter(configurations);
  // 触发自动化导入配置
  fireAutoConfigurationImportEvents(configurations, exclusions);
  return new AutoConfigurationEntry(configurations, exclusions);
}

```
