#核心组件
###综述
> OOP（面向对象编程）编程思想，那么Spring可以理解为BOP（面向Bean编程），即在Spring里面Bean是核心。

* Beans、Core、Context、SpEL
* Beans、Core 是框架的最基础备份，包含了IOC/DI
* Context 是建立在Beans、Core 之上，提供了框架式的访问方式
* SpEL 强大的运行时操作对象的表达式语言

###不恰当的类比
* 人民 Bean ，国家 Context ，工具 Core  
* Bean 是体现了以人为本的思想，Context 所有的人民都赖以生存的环境，Core 他们之间进行关系关联和管理的工具

###Bean 组件
>功能：Bean的创建，定义，解析

* 创建：DefaultListableBeanFactory 类结构图

	![](http://47.95.12.0:3389/ftp/DefaultListableBeanFactory.png)     
		
>上述的类结构图还是有点复杂的，其实这么设计原则，正是符合了单一责任原则，我们只需要关心从Beanfactory这条线到DefaultListableBeanFactory。它实现了上述所有接口和抽象类的功能,是一个正真的IOC容器。
	
* 定义：BeanDefinition 类结构图
	
  ![](http://47.95.12.0:3389/ftp/RootBeanDefinition.png)  
	
* 解析：XmlBeanDefinitionReader 类结构图
	
  ![](http://47.95.12.0:3389/ftp/XmlBeanDefinitionReader.png)  
  
  
###创建、定义、解析介绍  
>一般我们潜意识认为BeanFactory是一个IOC容器，当然它确实也是，但是我们需要注意它是一个接口，只是定义了一些规范，具体的实现有很多。BeanFactory默认的直接继承者有三个：HierarchicalBeanFactory ListableBeanFactory AutowireCapableBeanFactory 
 
* 一、创建 
	1. BeanFactory 
	   
		```java 
		public interface BeanFactory {
		    //主要是为了区分FactoryBean获得Bean是自身还是实现了它的对象
		    String FACTORY_BEAN_PREFIX = "&";
		
		    Object getBean(String var1) throws BeansException;
		    // 4个人不同入参的getBean方法
		    <T> T getBean(String var1, Class<T> var2) throws BeansException;
		
		    Object getBean(String var1, Object... var2) throws BeansException;
		
		    <T> T getBean(Class<T> var1) throws BeansException;
		
		    <T> T getBean(Class<T> var1, Object... var2) throws BeansException;
		
		    boolean containsBean(String var1);
		    //单例
		    boolean isSingleton(String var1) throws NoSuchBeanDefinitionException;
		    //原型
		    boolean isPrototype(String var1) throws NoSuchBeanDefinitionException;
		    //2个 类定义的名称和类型是否匹配方法
		    boolean isTypeMatch(String var1, ResolvableType var2) throws NoSuchBeanDefinitionException;
		
		    boolean isTypeMatch(String var1, Class<?> var2) throws NoSuchBeanDefinitionException;
		     //获取Class类型
		    Class<?> getType(String var1) throws NoSuchBeanDefinitionException;
		   //根据实例名字获取别名
		    String[] getAliases(String var1);
		}
		```   
	2. 直接继承者3个
	
		* 列出本工厂可以生产的Bean的工厂  ListableBeanFactory   
		
	      ``` java
		public interface ListableBeanFactory extends BeanFactory {
		    // 是否有Bean定义ByName
		    boolean containsBeanDefinition(String var1);
		    // 返回工厂里BeanDefinition总数量
		    int getBeanDefinitionCount();
		     // 返回工厂里所有Bean名字
		    String[] getBeanDefinitionNames();
		    // 3个根据类型获取BeanName方法
		    String[] getBeanNamesForType(ResolvableType var1);
		
		    String[] getBeanNamesForType(Class<?> var1);
		   
		    String[] getBeanNamesForType(Class<?> var1, boolean var2, boolean var3);
		     //2个根据类型（包括子类）返回指定Bean名和Bean的Map
		    <T> Map<String, T> getBeansOfType(Class<T> var1) throws BeansException;
		
		    <T> Map<String, T> getBeansOfType(Class<T> var1, boolean var2, boolean var3) throws BeansException;
		    //2个跟注解查找有关的方法
		    String[] getBeanNamesForAnnotation(Class<? extends Annotation> var1);
		
		    Map<String, Object> getBeansWithAnnotation(Class<? extends Annotation> var1) throws BeansException;
		
		    <A extends Annotation> A findAnnotationOnBean(String var1, Class<A> var2) throws NoSuchBeanDefinitionException;
		}
	     ```
	
	
	   * 分层工厂 HierarchicalBeanFactory
	   
			```java
			public interface HierarchicalBeanFactory extends BeanFactory {
			   //返回本Bean工厂的父工厂
				BeanFactory getParentBeanFactory();
				//本地工厂是否包含这个Bean
				boolean containsLocalBean(String name);
			
			}
		    ```
	
	   * 可以自动装配工厂 AutowireCapableBeanFactory
		
			```java
			public interface AutowireCapableBeanFactory extends BeanFactory {
			   // 5个常量，指明装配策略
				int AUTOWIRE_NO = 0;//没有自动装配的Bean
			
				int AUTOWIRE_BY_NAME = 1;//根据名称自动装配
			
				int AUTOWIRE_BY_TYPE = 2;//根据类型自动装配
			
				int AUTOWIRE_CONSTRUCTOR = 3;//根据构造方法自动装配
			
				@Deprecated
				int AUTOWIRE_AUTODETECT = 4;//通过Bean的class的内部来自动装配，废弃Since3.0
				/**
			 	* 8个跟自动装配有关的方法
			 	*/
				//指定Class创建一个全新的Bean实例
				<T> T createBean(Class<T> beanClass) throws BeansException;
				//根据Bean名的BeanDefinition装配这个未加工的Object，执行回调和各种后处理器。
				void autowireBean(Object existingBean) throws BeansException;
				//分解Bean在工厂中定义的这个指定的依赖descriptor
				Object configureBean(Object existingBean, String beanName) throws BeansException;
				//根据给定的类型和指定的装配策略，创建一个新的Bean实例
				Object createBean(Class<?> beanClass, int autowireMode, boolean dependencyCheck) throws BeansException;
			
				Object autowire(Class<?> beanClass, int autowireMode, boolean dependencyCheck) throws BeansException;
				//根据名称或类型自动装配
				void autowireBeanProperties(Object existingBean, int autowireMode, boolean dependencyCheck)
						throws BeansException;
			
				void applyBeanPropertyValues(Object existingBean, String beanName) throws BeansException;
			
				Object initializeBean(Object existingBean, String beanName) throws BeansException;
				//初始化之前执行BeanPostProcessors
				Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
						throws BeansException;
			   //初始化之后执行BeanPostProcessors
				Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
						throws BeansException; 
			
				void destroyBean(Object existingBean);
			
				<T> NamedBeanHolder<T> resolveNamedBean(Class<T> requiredType) throws BeansException;
			
			
				Object resolveDependency(DependencyDescriptor descriptor, String requestingBeanName) throws BeansException;
			   //分解指定的依赖
				Object resolveDependency(DependencyDescriptor descriptor, String requestingBeanName,
						Set<String> autowiredBeanNames, TypeConverter typeConverter) throws BeansException;
			
			}
			```
			
  * 说明：对于Spring来说最顶层的Bean工厂为BeanFactory，它定义的功能比较简单，getBean() 、 containsBean()等，并不关心资源、Bean的定义等信息。但它默认三个直接继承者扩展了它，Spring提供了蛮多种方式来生产Bean，比较繁杂，这些继承者们就是Spring分而治之这些难题的小组件。从类结构图可以看出DefaultListableBeanFactory是第一个普通类，它是一个集大成者，有完整的Bean生产能力。我们常知道的ClassPathXmlApllicationContext就是通过它的代理实现Bean工厂的。
 
* 二、定义  
  
  
  
  
  
  
  
  
  
  
  
  