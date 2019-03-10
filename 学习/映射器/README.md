# 映射器

> 映射器可以理解为我们平常说的mapper接口，一般映射器可以由接口+xml/注解方式组成。生命周期：一个会话内。

### 一、使用

* 接口+xml

```java
 @Mapper
 public interface userMapper{
 	User selectById(id);
 }
 <?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.vxiaoke.cp.base.mapper.UserAccountMapper">

    <resultMap id="BaseResultMap" type="com.vxiaoke.cp.base.models.UserAccount">
        <id column="id" property="id" />
        <result column="username" property="ctime" />
        <result column="age" property="mtime" />
    </resultMap>
    <!-- 通用查询结果列 -->
    <sql id="Base_Column_List">
        id, ctime, mtime, username, age
    </sql>

    <select id="selectById" resultMap="BaseResultMap">
        select <include refid="Base_Column_List"/>
        from user where id = #{id}
    </select>
 </mapper>
```
* 接口+注解

```java
 @Select(value="select *id,username,age from user where id=#{id}")
 public interface userMapper{
 	User selectById(id);
 }
``` 
### 二、组成
* MappedStatement 
* SqlSource
* BoundSql

### 三、说明
* MappedStatement
  这个类涉及的东西是比较多的，可以看看下面的表
  
|属性 | 类型|说明|
|---|---|---|----|
|resource | String| 类似mybatis-config.xml 文件名|
|Configuration | Configuration|配置类|
|String | id| 查找到哪个mapper标识，例如getById|
|KeyGenerator | keyGenerator| key生成，例如在insert后返回id|
|boolean | useCache| 是否用耳机缓存|
|SqlSource | sqlSource| 根据参数组装sql|
|ParameterMap | parameterMap|参数|
|... | ...| ...|
  
* SqlSource

> 根据参数/其他规则组装sql，

![](http://47.95.12.0:3389/ftp/SqlSource.png) 

```java 
public interface SqlSource {

  BoundSql getBoundSql(Object parameterObject);

}
```  

* BoundSql

```java 
public class BoundSql {

  private final String sql;//我们写的原生sql
  private final List<ParameterMapping> parameterMappings;
  private final Object parameterObject;//参数本身，例如POJO，Map等传入的参数，@Parm注解会解析成Map
  private final Map<String, Object> additionalParameters;//每个元素都是map，如：属性名，类型javaType，typeHandler等
  private final MetaObject metaParameters;

  public BoundSql(Configuration configuration, String sql, List<ParameterMapping> parameterMappings, Object parameterObject) {
    this.sql = sql;
    this.parameterMappings = parameterMappings;
    this.parameterObject = parameterObject;
    this.additionalParameters = new HashMap<>();
    this.metaParameters = configuration.newMetaObject(additionalParameters);
  }

  public String getSql() {
    return sql;
  }

  public List<ParameterMapping> getParameterMappings() {
    return parameterMappings;
  }

  public Object getParameterObject() {
    return parameterObject;
  }

  public boolean hasAdditionalParameter(String name) {
    String paramName = new PropertyTokenizer(name).getName();
    return additionalParameters.containsKey(paramName);
  }

  public void setAdditionalParameter(String name, Object value) {
    metaParameters.setValue(name, value);
  }

  public Object getAdditionalParameter(String name) {
    return metaParameters.getValue(name);
  }
}
```

* Mapper本质

```java
// MapperProxy工厂
public class MapperProxyFactory<T> {

  private final Class<T> mapperInterface;
  private final Map<Method, MapperMethod> methodCache = new ConcurrentHashMap<Method, MapperMethod>();

  public MapperProxyFactory(Class<T> mapperInterface) {
    this.mapperInterface = mapperInterface;
  }

  public Class<T> getMapperInterface() {
    return mapperInterface;
  }

  public Map<Method, MapperMethod> getMethodCache() {
    return methodCache;
  }

  @SuppressWarnings("unchecked")
  protected T newInstance(MapperProxy<T> mapperProxy) {
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
  }

  public T newInstance(SqlSession sqlSession) {
    final MapperProxy<T> mapperProxy = new MapperProxy<T>(sqlSession, mapperInterface, methodCache);
    return newInstance(mapperProxy);
  }

}

//MapperProxy JDK的动态代理，实现InvocationHandler 接口
public class MapperProxy<T> implements InvocationHandler, Serializable {

  private static final long serialVersionUID = -6424540398559729838L;
  private final SqlSession sqlSession;
  private final Class<T> mapperInterface;
  private final Map<Method, MapperMethod> methodCache;

  public MapperProxy(SqlSession sqlSession, Class<T> mapperInterface, Map<Method, MapperMethod> methodCache) {
    this.sqlSession = sqlSession;
    this.mapperInterface = mapperInterface;
    this.methodCache = methodCache;
  }
  
  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
    // 是否是个类，显然不是
      if (Object.class.equals(method.getDeclaringClass())) {
        return method.invoke(this, args);
      } else if (isDefaultMethod(method)) {
        return invokeDefaultMethod(proxy, method, args);
      }
    } catch (Throwable t) {
      throw ExceptionUtil.unwrapThrowable(t);
    }
    // 构建MapperMethod 对象，跳到下面*处
    final MapperMethod mapperMethod = cachedMapperMethod(method);
    // 核心方法（注意！）
    return mapperMethod.execute(sqlSession, args);
  }

// * 是否在缓存里，否则初始化mapperMethod
  private MapperMethod cachedMapperMethod(Method method) {
    MapperMethod mapperMethod = methodCache.get(method);
    if (mapperMethod == null) {
      mapperMethod = new MapperMethod(mapperInterface, method, sqlSession.getConfiguration());
      methodCache.put(method, mapperMethod);
    }
    return mapperMethod;
  }
...

//看看核心方法
public class MapperMethod {

  private final SqlCommand command;
  private final MethodSignature method;

  public MapperMethod(Class<?> mapperInterface, Method method, Configuration config) {
    this.command = new SqlCommand(config, mapperInterface, method);
    this.method = new MethodSignature(config, mapperInterface, method);
  }

// 命令模式,对底层还是sqlSession来执行。
  public Object execute(SqlSession sqlSession, Object[] args) {
    Object result;
    switch (command.getType()) {
      case INSERT: {
    	Object param = method.convertArgsToSqlCommandParam(args);
        result = rowCountResult(sqlSession.insert(command.getName(), param));
        break;
      }
      case UPDATE: {
        Object param = method.convertArgsToSqlCommandParam(args);
        result = rowCountResult(sqlSession.update(command.getName(), param));
        break;
      }
      case DELETE: {
        Object param = method.convertArgsToSqlCommandParam(args);
        result = rowCountResult(sqlSession.delete(command.getName(), param));
        break;
      }
      case SELECT:
        if (method.returnsVoid() && method.hasResultHandler()) {
          executeWithResultHandler(sqlSession, args);
          result = null;
        } else if (method.returnsMany()) {
          result = executeForMany(sqlSession, args);
        } else if (method.returnsMap()) {
          result = executeForMap(sqlSession, args);
        } else if (method.returnsCursor()) {
          result = executeForCursor(sqlSession, args);
        } else {
          Object param = method.convertArgsToSqlCommandParam(args);
          result = sqlSession.selectOne(command.getName(), param);//sqlSession
        }
        break;
      case FLUSH:
        result = sqlSession.flushStatements();
        break;
      default:
        throw new BindingException("Unknown execution method for: " + command.getName());
    }
    if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
      throw new BindingException("Mapper method '" + command.getName() 
          + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
    }
    return result;
  }

```

* 流程梳理
 
<center>![](http://sowcar.com/t6/679/1552231058x986907160.png)</center>
  