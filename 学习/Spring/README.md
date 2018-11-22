# Spring

> Spring 相信大家否不陌生，可以说在日常的开发中无处不在，自从2003年由[Rod Johnson](https://baike.baidu.com/item/Rod%20Johnson/1423612?fr=aladdin)创建以来在Java界掀起“春天大潮”，极大的解放了广大程序员。在此基础之上衍生出Spring很多框架，像SpringBoot，Spring JPA，SpingMVC等等。所有的这些都是基于Spring Framework 强大的基石。

* 高屋建瓴

![](http://wiki.jikexueyuan.com/project/spring/images/arch1.png)
  
* 核心容器层
  * Core 模块：提供了框架的基本组成部分，包括 IoC 及依赖注入功能。
  * Bean 模块：实现 Bean 管理，包括自动装配机制等功能； 其中BeanFactory是一个工厂模式的实现。

 * Context 模块：建立在 Core 和 Bean 模块基础上，通常用于访问配置及定义的任何对象。ApplicationContext 是上下文模块的重要接口。

 * SpEL 模块：表达式语言模块提供了运行时进行查询及操作一个对象的表达式机制。

* 数据访问/集成
	* JDBC 模块：用于替代繁琐的 JDBC API 的抽象层。

	* ORM 模块：对象关系数据库映射抽象层，可集成JPA，JDO，Hibernate，iBatis。

    * OXM 模块：XML消息绑定抽象层，支持JAXB，Castor，XMLBeans，JiBX，XStream。

   * JMS 模块：Java消息服务模块，实现消息生产-消费之类的功能。

  * Transaction 模块：事务模块为各种 POJO 支持编程式和声明式事务管理。

* Web应用
  * Web 模块：Web MVC 提供了基于 模型-视图-控制器 的基础web应用框架。

 * servlet 模块：实现了统一的监听器以及和面向web应用的上下文，用以初始化 IoC 容器。
  * Web-Portlet：实现在 portlet 环境中实现 MVC。

  * Web-Socket 模块：为 WebSocket连接 提供支持。
* AOP 模块

  * 提供了面向切面的编程实现，允许开发者通过定义方法拦截器及切入点对代码进行无耦合集成，它实现了关注点分离。

* 测试：支持 JUnit 、TestNG 框架的集成  