# 加载流程

> 我使用的springBoot框架，从它的入口方法main来分析。

```java
	@SpringBootApplication
    public class CollagePurchaseApplication {
        public static void main(String[] args) {
            SpringApplication.run(CollagePurchaseApplication.class, args);
        }
    }

    public static ConfigurableApplicationContext run(Object source, String... args) {
        return run(new Object[]{source}, args);
    }

    public static ConfigurableApplicationContext run(Object[] sources, String[] args) {
        return new SpringApplication(sources).run(args);
    }
    //⚠️ 核心方法
    public SpringApplication(Object... sources) {
        initialize(sources);
    }

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

>从以上的方法可以看出，主要的功能是开启一个SpringApplicationContext，即一个Spring的上下文（容器）。主要调用了initialize方法
  
  * 初始化方法initialize
     * deduceWebEnvironment()方法是验证是否有web运行的环境
     * getSpringFactoriesInstances方法
     
        ```java
      private <T> Collection<? extends T> getSpringFactoriesInstances(Class<T> type,Class<?>[] parameterTypes, Object... args) {
		//1.获取当前类加载器
		ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
		//2.加载所欲jar包下的META-INF/spring.factories
		Set<String> names = new LinkedHashSet<String>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
		//3.创建ApplicationContextInitializer.class 这个类型的实例
		List<T> instances = createSpringFactoriesInstances(type, parameterTypes,classLoader, args, names);
	   //4.将上述步骤3的实例排序
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
	}
	```
     






