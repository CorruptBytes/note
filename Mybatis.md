# 分页查询

原生`Mybatis`并没有内置分页查询功能。想要实现分页查询，有两三种方式：

- 手动拼接分页关键字与分页参数
- 分页插件
- `RowBounds`类

<h3>手动拼接</h3>

通过Java代码计算`offset`后手动拼接到`SQL`中。

```xml
int offset = (pageNum - 1) * pageSize;

<select id="selectUserPage" resultType="User">
    SELECT * FROM user
    ORDER BY id
    LIMIT #{offset}, #{pageSize}
</select>
```

**缺点**

- 需要在每个 SQL 中手动拼接分页关键字，如果替换数据库实现，可能需要修改所有分页语句。

<h3>分页插件</h3>

`Mybatis`允许通过`Interceptor`，在SQL执行过程中拦截并修改`SQL`，自动拼接分页关键字。这些`Interceptor`称为插件。存在一些第三方的分页插件：

- `PageHelper`

  ```xml
  <dependency>
      <groupId>com.github.pagehelper</groupId>
      <artifactId>pagehelper-spring-boot-starter</artifactId>
  </dependency>
  ```

- `MybatisPlus`内置分页插件

也可自己写`Interceptor`。

<h3><code>RowBounds</code></h3>

由`Mybatis`提供，但默认实现是**内存分页**，会先查出全部数据，再在内存中截取。如果数据量大，会直接导致内存爆炸。

```java
RowBounds rowBounds = new RowBounds(offset, pageSize);
```

# 插件机制

