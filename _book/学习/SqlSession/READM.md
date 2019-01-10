# SqlSession

* 源码

```java

public interface SqlSession extends Closeable {

  <T> T selectOne(String statement);

  <T> T selectOne(String statement, Object parameter);

  <E> List<E> selectList(String statement);

  <E> List<E> selectList(String statement, Object parameter);

  <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds);

  <K, V> Map<K, V> selectMap(String statement, String mapKey);

  <K, V> Map<K, V> selectMap(String statement, Object parameter, String mapKey);

  <K, V> Map<K, V> selectMap(String statement, Object parameter, String mapKey, RowBounds rowBounds);

  <T> Cursor<T> selectCursor(String statement);

  <T> Cursor<T> selectCursor(String statement, Object parameter);

  <T> Cursor<T> selectCursor(String statement, Object parameter, RowBounds rowBounds);
  
  void select(String statement, Object parameter, ResultHandler handler);

  void select(String statement, ResultHandler handler);

  void select(String statement, Object parameter, RowBounds rowBounds, ResultHandler handler);

  int insert(String statement);

  int insert(String statement, Object parameter);

  int update(String statement);

  int update(String statement, Object parameter);

  int delete(String statement);

  int delete(String statement, Object parameter);

  void commit();

  void commit(boolean force);

  void rollback();

  void rollback(boolean force);

  List<BatchResult> flushStatements();

  @Override
  void close();

  void clearCache();
  
  Configuration getConfiguration();

  <T> T getMapper(Class<T> type);

  Connection getConnection();
}
```


* 默认实现

![](http://47.95.12.0:3389/ftp/SqlSession.png) 

>SqlSession定义了有很多查询数据的方法，因此我们也可以直接使用它来操作为业务sql执行器，但是这样做不是很好。一般情况DefaultSqlSession作为默认的实现，SqlSessionManager 从项目源码来看也没有地方被引用，只有它test类。