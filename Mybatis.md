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

# mybatis中对业务层传入的大集合的处理方法

1. 临时表
   将列表先批量插入临时表，再用 JOIN 关联业务表查询。临时表是会话级的，并发安全。

```
<!-- 1. 创建临时表 -->
<update id="createIdTempTable">
  CREATE TEMPORARY TABLE IF NOT EXISTS temp_id_list (
    id BIGINT NOT NULL,
    PRIMARY KEY (id)
  ) ENGINE = MEMORY
</update>

<!-- 2. 批量插入（SQL 非常短，结构固定） -->
<insert id="batchInsertIds">
  INSERT INTO temp_id_list (id) VALUES
  <foreach collection="list" item="id" separator=",">
    (#{id})
  </foreach>
</insert>

<!-- 3. JOIN 查询（SQL 简洁，性能极佳） -->
<select id="selectByIdsJoinTemp" resultType="User">
  SELECT u.* 
  FROM user u
  INNER JOIN temp_id_list t ON u.id = t.id
</select>
```

2. JSON_TABLE
   将列表序列化为一个 JSON 数组字符串传入，用 JSON_TABLE 函数直接展开成表。

```
<select id="selectByIdsJsonTable" resultType="User">
  SELECT u.*
  FROM user u
  INNER JOIN JSON_TABLE(
    #{idJson}, '$[*]' COLUMNS (id BIGINT PATH '$')
  ) AS jt ON u.id = jt.id
</select>
```

1. 字符串拆分函数
   将列表拼接成一个逗号分隔的长字符串，用数据库内置的拆分函数转成表。
2. MyBatis 批处理 + 应用层分片
   foreach标签配合应用层分片

# N + 1 查询问题

在获取一组数据时，执行1次主查询获取N条记录，然后为获取关联数据再对每条记录单独执行1次关联查询，总共产生N+1次数据库交互，导致性能严重下降。

- 子查询导致的 N 次查询不属于N + 1 问题，因为此时只有一次数据库交互

# Mybatis缓存机制
Mybatis中存在两级缓存：
- **一级缓存**：默认为`SqlSession` 级别。在`Mybatis`中称为本地缓存(`localCache`)，默认开启。
- **二级缓存**：`Mapper Namespace`级别，可跨`SqlSession`，需要显示开启。

查询时，优先查询二级缓存(如果开启)，未命中再查询一级缓存，均不存在才会查询数据库。

```
Mapper 查询
   ↓
CachingExecutor
   ↓
二级缓存
   │
   ├─ 命中 → 直接返回
   │
   └─ 未命中
        ↓
   BaseExecutor.query()
        ↓
   一级缓存 localCache
        │
        ├─ 命中 → 返回
        │
        └─ 未命中
             ↓
          查询数据库
```

## 一级缓存
一级缓存为`localCache`，默认作用域是 `SqlSession`，不同 `SqlSession` 互不共享。
<h4>一级缓存的 <code>Key</code></h4>
一级缓存的 `Key` 为 Mybatis自定义对象 `CacheKey`，它包含以下信息：

```
MappedStatement ID
+
分页 offset
+
分页 limit
+
最终 SQL
+
SQL 参数
+
Environment ID
```

<h4>一级缓存失效场景</h4>

一级缓存在以下情况下会失效：
- **`SqlSession` 关闭：**一级缓存属于 `SqlSession` ， `SqlSession` 关闭，一级缓存自然消失。
- **执行增删改： **`Mybatis` 采用比较直接的缓存更新策略，只要执行了增删改操作，直接清空当前 `SqlSession` 的一级缓存。
- **手动执行 `clearCache()`：**`SqlSession`提供了手动清理一级缓存的接口`clarCache()`，调用方法会直接清空一级缓存。

<h4>一级缓存作用域配置</h4>

一级缓存的默认作用域为 `SqlSession`， `Mybatis`中提供了一个配置项用于配置一级缓存作用域：

```
<setting name="localCacheScope" value="SESSION"/>
```

作用域选项包含：

- **`SESSION`：**默认作用域，一级缓存在一个`SqlSession`实例内生效。
- **`STATEMENT`：**一级缓存只在一次语句执行过程中生效，语句执行结束立即清除。

实际开发里通常保持默认的 `SESSION`。

## 二级缓存

一级缓存只能被同一个 `SqlSession`实例共享，因此 `Mybatis` 又提供了二级缓存，它的作用域为`Mapper Namespace`，可以跨 `SqlSession`共享。

```xml
<mapper namespace="com.example.mapper.UserMapper">
```

整个 `namespace` 下共享二级缓存。

<h4>开启二级缓存</h4>

首先需要在全局 `cacheEnabled` 开启允许二级缓存；它默认就是开启的。

然后在 `Mapper XML`中为需要的 `Mapper` 开启二级缓存：

```xml
<mapper namespace="com.example.mapper.UserMapper">
    
    <cache/>

    <select id="selectById"
            resultType="User">
        SELECT * FROM user WHERE id = #{id}
    </select>

</mapper>
```

`<cache/>` 表示为这个 namespace 开启二级缓存。

<h4>二级缓存生效时机</h4>

二级缓存不会在查询结束后立刻生效，通常需要：

```java
session.commit();
//或
session1.close();
```

相关缓存事务完成后，数据才对其他 `SqlSession` 生效。

<h4>二级缓存数据一致性问题</h4>

执行更新时，对应 `Mapper Namespace` 的缓存会被清除。但是复杂 `SQL` 通常会涉及多张表，二级缓存基于 `Mapper Namespace` 维度管理，而不是数据库表维度。

因此在复杂关联查询场景下，缓存的一致性管理会非常麻烦。

## 两级缓存实现原理

`Mybatis`中的两级缓存分别通过两个 `Executor` 实现，其中一级缓存由`BaseExecutor`实现；二级缓存通过装饰器模式实现，由 `CachingExecutor` 对底层 `Executor`装饰实现。