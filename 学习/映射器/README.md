# 映射器

> 映射器可以理解为我们平常说的mapper接口，一般映射器可以由接口+xml/注解方式组成。生命周期：一个会话内。

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
