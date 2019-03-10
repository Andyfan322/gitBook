# 四大金刚-StatementHandler

> 四大金刚中的核心，使用数据库的PreparedStatement执行数据库操作。

* StatementHandler 数据库会话器

```java
public interface StatementHandler {

  Statement prepare(Connection connection, Integer transactionTimeout)
      throws SQLException;

  void parameterize(Statement statement)
      throws SQLException;//ParameterHandler 参数设置类

  void batch(Statement statement)
      throws SQLException;

  int update(Statement statement)
      throws SQLException;

  <E> List<E> query(Statement statement, ResultHandler resultHandler)
      throws SQLException;//ResultSetHandler 结果集处理类

  <E> Cursor<E> queryCursor(Statement statement)
      throws SQLException;

  BoundSql getBoundSql();

  ParameterHandler getParameterHandler();

}
```

* 引出另外2大金刚
   * ParameterHandler
   * ResultSetHandler
