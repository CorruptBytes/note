# 概述

> 在 MyBatis 基础上进行功能扩展的增强工具，旨在不改变MyBatis原有功能和使用方式的前提下,简化开发，提高效率。

## 特性

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，还可以通过条件构造器，快速实现复杂查询。
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通查询

## 相关依赖

**Mybatis-Plus**

```xml
<dependency>
	<groupId>com.baomidou</groupId>
	<artifactId>mybatis-plus</artifactId>
	<version>3.5.5</version>
</dependency>
```

- MyBatis-Plus 的核心依赖包
- 依赖内部已经引入了Mybatis，引入后不需要再引入mybatis的依赖

**MybatisPlus起步依赖**

```xml
<!--Springboot3.X使用mybatis-plus-spring-boot3-starter -->
<dependency>
	<groupId>com.baomidou</groupId>
	<artifactId>mybatis-plus-spring-boot3-starter</artifactId>
	<version>3.5.5</version>
</dependency>
<!--Springboot2.X使用 mybatis-plus-boot-starter -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.12</version>
</dependency>
```

- 配合springboot使用的起步依赖包，实现了自动配置等功能

- 依赖内部已经集成了Mybatis和MybatisPlus的相关功能和自动装配功能,引入后无需再引入其他Mybatis相关依赖。

# 注解

## 实体类相关

MP遵循约定大于配置的规则将实体类映射到表结构中，但如果表结构与实体类不符合约定，可以通过注解手动配置映射关系。

**`@EnumValue`**

```java
  @TableName("student")
  class Student {
      private Integer id;
      private String name;
      private GradeEnum grade;//数据库grade字段类型为int
  }
 
  public enum GradeEnum {
      PRIMARY(1,"小学"),
      SECONDORY("2", "中学"),
      HIGH(3, "高中");
 
      @EnumValue
      private final int code;
      private final String descp;
  }
```

- 在枚举类成员字段上使用
- 当实体类的成员字段类型为枚举类时，会自动将枚举类中具有`@EnumValue`注解的成员字段的值映射为数据库字段的值

- @TableName

  ```java
  @TableName("tb_user")
  public class User {
  	private Long id;
  	private String username;
  	private String password;
      private String phone;
      private String info;
      private Integer status;
  }
  ```

  - 指定实体类对应的表名

  - 默认约定表名为实体类名驼峰转下划线

    

    

    TableId

  ```java
  public class User {
  	@TableId(value="id",type=Idtype.AUTO)
  	private Long id;
  	private String username;
  	private String password;
      private String phone;
      private String info;
      private Integer status;
  }
  ```

  - 指定主键字段名并配置主键的相关行为。
    - 默认约定表的主键字段为`"id","Id","ID"`中的一个
  - value指定表中的主键字段名，type为主键的生成策略，值为IdType的枚举类型:
    - AUTO：自增长
    - INPUT：通过set方法自行输入
    - ASSIGN_ID：分配 ID，接口`IdentifierGenerator`的方法nextId来生成id，默认实现类为DefaultIdentifierGenerator雪花算法，默认生成策略
  - 当使用`AUTO`这种不由自己指定ID的策略插入数据时，MP会将ID回写到实体类中(默认只有调用MP在`BaseMapper`中定义好的`insert`语句才生效，自己编写的`insert`语句需要通过特殊配置才可以回写id)

- @TableField

  ```java
  public class User {
  	private Long id;
  	private String username;
      @TableField("is_married")
  	private Boolean isMarried;
      @TableField("`order`")
      private Integer order;
      @TableField(exist = false)
      private String address;
  }
  ```

  - 指定成员字段对应的表中的普通字段名
  - 默认成员字段名与数据库字段名一致，如果开启了驼峰转下划线映射，则会将驼峰命名的成员字段名转为下划线命名对应到表字段名。
  - 使用场景:
    - 成员变量名与数据库字段名不一致
    - 成员变量名以is开头，且是布尔值,默认反射时会将`is`去掉作为字段名
    - 成员变量名与数据库关键字冲突,要用反引号包裹指明字段名
    - 成员变量不是数据库字段，通过exist属性标明


**`@Alias`**

```java
@Data
@AllArgsConstructor
@Alias("carousel")
public class Carousel {
    @TableId("")
    private Long carousel_id;
    private String img_path;
    private String describes;
}
```

- 为实体类指定别名,之后在`Mapper.XML`文件中可以直接使用别名指定实体类而不是全类名

## Bean相关

**`@Mapper`**

```java
@Mapper
public interface UserMapper{
}
```

- 标记在 **Mapper 接口** 上，告诉 MyBatis这是一个 Mapper，Spring 会生成其实体类并注册为 Bean。

**`@MapperScan`**

```java
@MapperScan("mapper包名")
@SpringBootApplication
public class MpDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(MpDemoApplication.class, args);
    }
}
```

- 标记在 **配置类（通常是启动类）** 上，指定一个或多个包路径，自动扫描这些包里的所有 Mapper 接口并注册为Bean。

# 配置

> **MyBatisPlus**的配置项继承了MyBatis原生配置并扩展了一些自己特有的配置,在mybatis-plus节点下进行配置，会同时作用于Mybatis。

```yaml
mybatis-plus:
  type-aliases-package: com.itheima.mp.domain.po # 指定一个或多个包路径(多个用,隔开)，自动扫描这些包下的所有 Java 类并为它们创建 别名(type alias),之后在 Mapper XML 文件中直接使用类名（而不用全限定名）默认别名 = 类名（区分大小写取决于配置）
  mapper-locations: "classpath*:/mapper/**/*.xml" # Mapper.xml文件地址，默认值
  configuration:
	map-underscore-to-camel-case: true # 开启类成员属性驼峰命名到表字段下划线命名的映射,默认关闭
    cache-enabled: false # 是否开启二级缓存
  global-config:
    db-config:
    id-type: assign_id # id为雪花算法生成
    update-strategy: not_null # 更新策略：只更新非空字段
```

|                                                           |                                                        |     可选值      |               说明                |
| :-------------------------------------------------------: | :----------------------------------------------------: | :-------------: | :-------------------------------: |
| `mybatis-plus.configuration.map-underscore-to-camel-case` | 是否开启实体类成员变量驼峰命名到表字段下划线命名的映射 | `true`  `false` | 继承自`mybatis`的配置项，默认关闭 |
|                                                           |                                                        |                 |                                   |
|                                                           |                                                        |                 |                                   |
|                                                           |                                                        |                 |                                   |

# 通用Mapper

  > MyBatis-Plus 提供了 `BaseMapper<T>` 接口，定义了常见的SQL映射方法。自定义 Mapper 只需继承该接口，即可在不编写任何 SQL 的情况下获得基本的 CRUD 功能。

## 基本用法

```java
//泛型中为被操作表对应的实体类
@Mapper
public interface UserMapper extends BaseMapper<User> {
}
```

- 继承通用`Mapper`并不影响通过注解或XML文件在自定义Mapper中编写自定义SQL。

## 成员方法

|                                         |                                        |      |
| --------------------------------------- | -------------------------------------- | ---- |
| List<T>  selectList(Wrapper<T> wrapper) | 根据条件构造器查询，返回一个实体类列表 |      |
|                                         |                                        |      |
|                                         |                                        |      |

## 原理

> MP通过扫描泛型中的实体类，基于反射获取实体类信息作为数据库信息

```java
@Data
public class User {
	private Long id;
	private String username;
	private String password;
    private String phone;
    private String info;
    private Integer status;
    private Integer balance;
    private LocalDateTime createTime;
    private LocalDateTime updateTime;
}

```

**默认MybatisPlus的转化规则为:**

- 类名驼峰转下划线作为表名
- 名为id的成员字段作为主键,对应表中名为`"id","Id","ID"`的主键字段。
- 变量名直接作为字段名；如果开启驼峰转下划线配置，则变量名驼峰转下划线作为表的字段名

# 通用Service

> MyBatis-Plus 提供了通用的 Service 层接口与实现类。自定义 Service 通过继承这些接口和类，能够直接使用通用的业务逻辑方法，无需重复编写基础代码。

## 基本用法

1. 创建接口，继承`IService<T>`接口,泛型为此Service对应的实体类

   ```
   public interface ICarouselService extends IService<Carousel> {
   }
   ```

2. 创建接口的实现类，并继承`ServiceImpl<M,T>`,M为此Service依赖的Mapper，T为对应的实体类。

   ```
   @Service
   public class CarouselServiceImpl extends ServiceImpl<CarouselMapper, Carousel> implements ICarouselService {
   
   }
   ```


# 条件构造器

> **条件构造器（Wrapper）** 是 MyBatis-Plus 提供的一种用于 **构建动态 SQL 条件语句** 的工具类。 它通过**链式调用**的方式来拼接各种 SQL 条件（如 `WHERE`、`ORDER BY`、`LIMIT` 等），可与 **通用 Mapper** 或 **通用 Service** 中的方法配合使用， 从而在不手写 SQL 的情况下实现灵活的数据查询与更新。

## 常用条件构造器

|          构造器          |                    说明                     |
| :----------------------: | :-----------------------------------------: |
|    `QueryWrapper<T>`     |      用于构造查询条件，最常用的Wrapper      |
|    `UpdateWrapper<T>`    | 用于构造更新条件,是`QueryWrapper<>`的扩展。 |
| `LambdaQueryWrapper<T>`  |       使用 Lambda 表达式构造查询条件        |
| `LambdaUpdateWrapper<T>` |       使用 Lambda 表达式构造更新条件        |

- 条件构造器的泛型均指定为查询结果对应的实体类。
- 一般除更新语句外，均使用`QueryWrapper`

![image-20250827125519544](D:\笔记\图片\image-20250827125519544.png)

**通用方法**

> 共有方法均继承自`AbstractWrapper`,主要是构造`where`条件的方法

| 方法                          | 作用              | 示例                                              |
| ----------------------------- | ----------------- | ------------------------------------------------- |
| `eq(column, val)`             | 等于              | `eq("name", "Tom")`                               |
| `ne(column, val)`             | 不等于            | `ne("status", 0)`                                 |
| `gt(column, val)`             | 大于              | `gt("age", 18)`                                   |
| `ge(column, val)`             | 大于等于          | `ge("score", 60)`                                 |
| `lt(column, val)`             | 小于              | `lt("age", 30)`                                   |
| `le(column, val)`             | 小于等于          | `le("price", 1000)`                               |
| `between(column, val1, val2)` | BETWEEN 条件      | `between("age", 18, 30)`                          |
| `like(column, val)`           | 模糊匹配          | `like("name", "Tom")`                             |
| `notLike(column, val)`        | 反向模糊匹配      | `notLike("name", "test")`                         |
| `in(column, values)`          | IN 条件           | `in("id", list)`                                  |
| `notIn(column, values)`       | NOT IN 条件       | `notIn("id", list)`                               |
| `isNull(column)`              | IS NULL           | `isNull("email")`                                 |
| `isNotNull(column)`           | IS NOT NULL       | `isNotNull("email")`                              |
| `orderByAsc(columns…)`        | 升序排序          | `orderByAsc("age")`                               |
| `orderByDesc(columns…)`       | 降序排序          | `orderByDesc("create_time")`                      |
| `last(sql)`                   | 拼接原生 SQL 片段 | `last("LIMIT 1")`                                 |
| `or()` / `and()`              | 逻辑连接          | `or().eq("status", 1)`                            |
| `nested(Consumer<Wrapper>)`   | 嵌套条件          | `nested(w -> w.eq("age", 18).or().eq("age", 20))` |

**QueryWrapper特殊方法**

> 支持控制查询字段、分组等

| 方法                                          | 作用             | 示例                                                         |
| --------------------------------------------- | ---------------- | ------------------------------------------------------------ |
| `select(columns…)`                            | 指定查询字段     | `select("id", "name")`                                       |
| `select(Class<T>, Predicate<TableFieldInfo>)` | 通过条件选择字段 | `select(User.class, info -> !info.getColumn().equals("password"))` |
| `groupBy(columns…)`                           | 分组             | `groupBy("dept_id")`                                         |
| `having(sqlSegment)`                          | HAVING 条件      | `groupBy("dept_id").having("COUNT(id) > 5")`                 |

- select

  > 设置查询字段

  ```
  QueryWrapper<User> wrapper = new QueryWrapper<User>().select("id","username","info","balance");
  ```

  |       重载方法参数        |                      |      |
  | :-----------------------: | :------------------: | :--: |
  | select(String... columns) | 查询字符串对应的字段 |      |
  |                           |                      |      |
  |                           |                      |      |

**UpdateWrapper特殊方法**

> 扩展了用于设置更新字段的功能。

| 方法                 | 作用                  | 示例                          |
| -------------------- | --------------------- | ----------------------------- |
| `set(column, val)`   | 设置更新字段          | `set("status", 1)`            |
| `setSql(sqlSegment)` | 拼接原生 SQL 更新片段 | `setSql("score = score + 1")` |

**LambdaWrapper**

> 相比于普通的Wrapper通过硬编码字符串指定字段名，它支持通过解析方法引用来指定字段。其余用法均与普通Wrapper一致。

```java
LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
wrapper.select(User::getId);
```

## 自定义SQL

> 通过Wrapper构建`Where`条件，剩下的部分自己定义

![image-20250827140308626](D:\笔记\图片\image-20250827140308626.png)

# 扩展功能

## 动态表名插件

> 可以在运行时动态地改变 SQL 语句中的表名，而不是固定为与实体类绑定的表名

**原理**

通过`DynamicTableNameInnerInterceptor`拦截器在SQL语句执行前拦截，根据配置的规则动态的替换表名。

### 快速使用

1. 配置拦截器

   ```java
       @Bean
       public DynamicTableNameInnerInterceptor dynamicTableNameInnerInterceptor () {
           Map<String, TableNameHandler> map = new HashMap<>(1);
           map.put("points_board", new TableNameHandler() {
               @Override
               public String dynamicTableName(String sql, String tableName) {
                   return TableInfoContext.getInfo();
               }
           });
           return new DynamicTableNameInnerInterceptor(map);
       }
   ```

   - sql执行前,会将map的键与表名比较，如果符合，则使用`TableNameHandler`的返回值替换表名

2. 将拦截器添加到MyBatisPlus总拦截器中。

## 分页查询插件

> 在MybatisPlus中，分页查询通过分页拦截器(`PaginationInnerInterceptor`)实现。

- MybatisPlus提供了多种拦截器实现了不同的功能，这些拦截器也被称为插件

**`PaginationInnerInterceptor`**

> 分页拦截器,内部具有一些属性，会影响拦截时的行为

**成员属性**

|                 |                                        |      |
| :-------------: | :------------------------------------: | ---- |
| `Long maxLimit` |         最大的单页分页条数限制         |      |
| `DbType dbType` | 数据库的类型，不同的数据库分页语句不同 |      |

### 原理

1. `PaginationInnerInterceptor`拦截 `Executor.query()` 方法（也就是执行 SQL 查询的地方）
2. 读取原始的查询语句,并根据数据库类型拼接分页语句
3. 执行拼接后的SQL语句并返回结果

### 基本用法

1. 在MybatisPlus配置类中创建拦截器容器`Bean`,并在容器中注册分页拦截器，开启分页功能

   ```java
   @Configuration
   public class MybatisPlusConfig {
   
       @Bean
       public MybatisPlusInterceptor mybatisPlusInterceptor() {
           MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
           // 添加分页拦截器，并指定数据库类型
           interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
           return interceptor;
       }
   }
   ```

   - `MybatisPlusInterceptor`是拦截器的容器，可以添加多个功能不同的拦截器

2. 创建`Page`对象，指定分页参数

   ```java
           int pageNum = 1,pageSize = 8;
           Page<Product> page = Page.of(pageNum,pageSize);
           LambdaQueryWrapper<Product> wrapper = new LambdaQueryWrapper<>();
           wrapper.orderBy(true,false,Product::getProductSales);
           List<Product> products = productService.list(page, wrapper);
   ```

**Service提供了多种方法通过`Page`进行分页查询**

- `page`方法

  ```
  Page page(Page page)
  
  Page page(Page page, Wrapper<T> queryWrapper)
  ```

  - 这一方法将查询结果封装到传入的`Page`对象中，通过`getRecords`方法获得结果
  - `Page`对象中除了结果，还封装了查询的相关信息，如总条数以及总页数

- `list`方法

  ```
  List<T> list(IPage<T> page, Wrapper<T> queryWrapper)
  ```

  - 直接根据分页条件查询并返回结果

## 逻辑删除

> 一些数据库中的数据是非常重要的，即使有删除数据的需求，也不能真正的删除，通常会为数据添加一个新字段标记是否被删除(一般1标识删除，0标识未删除),这称为逻辑删除。

- 此时在执行删除数据的SQL语句时，通过`Update`语句而并非是`Delete`语句。

  ```sql
  UPDATE user SET deleted = 1 WHERE id = 1 AND deleted = 0
  ```

- 注意查询时也要保证查询到的是未删除的数据

  ```sql
  SELECT * FROM user WHE	RE deleted = 0
  ```

### 使用

> MP支持通过全局配置和注解配置开启逻辑删除功能。

**全局配置**

```yaml
mybatis-plus:
  global-config:
    db-config:      
      logic-delete-field: flag # 全局逻辑删除的实体字段名，字段类型可以是boolean、integer
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
```

- 配置逻辑删除后，在执行相关实体类的查询与删除语句时，会自动按照逻辑删除的方式进行SQL操作
- 使用逻辑删除会导致数据库中的无用数据增多，影响效率。可以将被删除的数据迁移到其他表中来代替逻辑删除。

**注解配置**

```java
    @TableLogic(value = "0", delval = "1") // <--- 直接通过注解定义逻辑删除规则
    private Integer deleted;
```

- **注解优先级高于全局配置**。

| 属性名   | 含义                                                |
| -------- | --------------------------------------------------- |
| `value`  | 未删除时字段的值（相当于 `logic-not-delete-value`） |
| `delval` | 删除时字段的值（相当于 `logic-delete-value`）       |

## 代码生成器

> MybatisPlus提供的插件，可以根据数据库表信息生成对应的Java实体类以及Mapper，Service和Controller

### 使用

> 代码生成器通过Idea的插件使用，插件名为`MybatisPlus`

1. 在Idea`Tool`选项下选择`Config Database`配置好数据库连接
2. 在`Idea`的`Tool`选项下选择`Code Generator`配置所需生成的表以及Java类所在的包