# 加载流程
### springBoot大致启动流程

```java
   //1.我的加载main方法
	@SpringBootApplication
    public class CollagePurchaseApplication {
        public static void main(String[] args) {
            SpringApplication.run(CollagePurchaseApplication.class, args);
        }
    }
   //2.看看里面run方法
    public static ConfigurableApplicationContext run(Object source, String... args) {
        return run(new Object[]{source}, args);
    }
   //3.run真正被调用的
    public static ConfigurableApplicationContext run(Object[] sources, String[] args) {
        return new SpringApplication(sources).run(args);
    }
    //4.⚠️ 调用链里核心方法
    public SpringApplication(Object... sources) {
        initialize(sources);
    }
  //5.初始化方法
    @SuppressWarnings({"unchecked", "rawtypes"})
    private void initialize(Object[] sources) {
        if (sources != null && sources.length > 0) {
            this.sources.addAll(Arrays.asList(sources));
        }
        this.webEnvironment = deduceWebEnvironment();
        setInitializers((Collection) getSpringFactoriesInstances(
                ApplicationContextInitializer.class));
        setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
        this.mainApplicationClass = deduceMainApplicationClass();
    }
```

* 分析

>从以上的方法可以看出，主要的功能是开启一个SpringApplicationContext，即一个Spring的上下文（容器）。主要调用了run
方法既而里面initialize方法是核心。

  * 初始化方法initialize
     * deduceWebEnvironment()方法是验证是否有web运行的环境
     * getSpringFactoriesInstances方法
 
  ```java
      private <T> Collection<? extends T> getSpringFactoriesInstances(Class<T> type,Class<?>[] parameterTypes, Object... args) {
		//1.获取当前类加载器
		ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
		//2.加载所在jar包下的META-INF/spring.factories
		Set<String> names = new LinkedHashSet<String>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
		//3.创建ApplicationContextInitializer.class 这个类型的实例
		List<T> instances = createSpringFactoriesInstances(type, parameterTypes,classLoader, args, names);
	   //4.将上述步骤3的实例排序
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
	}
	```
* run方法
    * 创建配置环境
    * 事件监听
    * 创建应用上下文
    * 实现自动化配置
  
   ```java
   public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		FailureAnalyzers analyzers = null;
		configureHeadlessProperty();
		//创建应用监听器，开始监听
		SpringApplicationRunListeners listeners = getRunListeners(args);
		listeners.starting();
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
			//加载SpringBoot的配置环境
			ConfigurableEnvironment environment = prepareEnvironment(listeners,applicationArguments);
			Banner printedBanner = printBanner(environment);
			//创建配置上下文
			context = createApplicationContext();
			analyzers = new FailureAnalyzers(context);
			//将listeners、environment、applicationArguments、banner等重要组件与上下文对象关联
			prepareContext(context, environment, listeners, applicationArguments,printedBanner);
			refreshContext(context);
			afterRefresh(context, applicationArguments);
			//实现spring-boot-starter-*(mybatis、redis等)自动化配置的关键，包括spring.factories的加载，bean的实例化等核心工作。
			listeners.finished(context, null);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
			}
			return context;
		}
		catch (Throwable ex) {
			handleRunFailure(context, listeners, analyzers, ex);
			throw new IllegalStateException(ex);
		}
	}
``` 






