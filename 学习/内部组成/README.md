# 内部组成

##### 一、组成
* MappedStatement 
* SqlSource
* BoundSql

##### 二、说明
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

  private final String sql;
  private final List<ParameterMapping> parameterMappings;
  private final Object parameterObject;
  private final Map<String, Object> additionalParameters;
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
  