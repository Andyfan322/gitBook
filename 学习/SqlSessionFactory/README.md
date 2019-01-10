# SqlSessionFactoryBuilder
> 这个类是用来构建SqlSessionFactory工厂用的，用完即回收。生命周期：局部方法内。

### 类方法

![](http://47.95.12.0:3389/ftp/SqlSessionFactoryBuilder.png) 

### 方法
> 前8个方法都是从XMl里面构建SqlSessionFactory，最后一个方法是从Configuration对象来构建SqlSessionFactory。

* 使用XML方式构建	mybatis-config.xml	


``` xml
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
"http://mybatis.org/dtd/mybatis-3-config.dtd">  

<configuration>   
    <!-- 加载类路径下的属性文件 -->  
    <properties resource="db.properties"/>  
    <!-- 设置类型别名 -->  
    <typeAliases>  
        <typeAlias type="cn.itcast.javaee.mybatis.app04.Student" alias="student"/>  
    </typeAliases>  
    <!-- 设置一个默认的连接环境信息 -->  
    <environments default="mysql_developer">  
        <!-- 连接环境信息，取一个任意唯一的名字 -->  
        <environment id="mysql_developer">  
            <!-- mybatis使用jdbc事务管理方式 -->  
            <transactionManager type="jdbc"/>  
            <!-- mybatis使用连接池方式来获取连接 -->  
            <dataSource type="pooled">  
                <!-- 配置与数据库交互的4个必要属性 -->  
                <property name="driver" value="${mysql.driver}"/>  
                <property name="url" value="${mysql.url}"/>  
                <property name="username" value="${mysql.username}"/>  
                <property name="password" value="${mysql.password}"/>  
            </dataSource>  
        </environment>  
        <!-- 连接环境信息，取一个任意唯一的名字 -->  
        <environment id="oracle_developer">  
            <!-- mybatis使用jdbc事务管理方式 -->  
            <transactionManager type="jdbc"/>  
            <!-- mybatis使用连接池方式来获取连接 -->  
            <dataSource type="pooled">  
                <!-- 配置与数据库交互的4个必要属性 -->  
                <property name="driver" value="${oracle.driver}"/>  
                <property name="url" value="${oracle.url}"/>  
                <property name="username" value="${oracle.username}"/>  
                <property name="password" value="${oracle.password}"/>  
            </dataSource>  
        </environment>  
    </environments>  
    <!-- 加载映射文件-->  
    <mappers>  
        <mapper resource="cn/itcast/javaee/mybatis/app14/StudentMapper.xml"/>  
    </mappers>  
</configuration>  

```

```java 
//伪代码获取SqlSessionFactory
String resource="mybatis-config.xml";
InputStream is=Resource.getResourceAsStream(source);
SqlSessionFactory sqlSessionFactory=new SqlSessionFactoryBuilder.build(is);
```

*  使用Java代码构建

```java
DataSource dataSource = BlogDataSourceFactory.getBlogDataSource();
TransactionFactory transactionFactory = new JdbcTransactionFactory();
Environment environment = new Environment("development", transactionFactory, dataSource);
Configuration configuration = new Configuration(environment);
configuration.addMapper(BlogMapper.class);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
```
