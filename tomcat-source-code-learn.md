#### 什么是tomcat

The Apache Tomcat® software is an open source implementation of the Jakarta Servlet, Jakarta Server Pages, Jakarta Expression Language, Jakarta WebSocket, Jakarta Annotations and Jakarta Authentication specifications. These specifications are part of the Jakarta EE platform.

Apache Tomcat® 软件是 Jakarta Servlet、Jakarta Server Pages、Jakarta Expression Language、Jakarta WebSocket、Jakarta Annotations 和 Jakarta Authentication 规范的开源实现。 这些规范是 Jakarta EE 平台的一部分。

tomcat 是一个容器， 用来承载servlet， tomcat其实就是一个实现了部分J2EE规范的服务器

#### 什么是规范

Java 是一门语言， 那么语言是不是需要依赖第三方对于他们自己产品的实现， 以一种jar包的形式提供给我们直接调用， 那么这个jar包称之为： SDK

#### Tomcat中的web.xml

所有的web应用， 它们的默认值都取决于conf/web.xml, 你配置的web.xml的信息， 是可以覆盖的。可以类比于类的继承和方法的复写

#### Tomcat声明周期

1. init
2. start
3. stop
4. destroy
