# 概述

> Google推出的项目构建与依赖管理工具，使用Groovy编写。

**特点**

- 约定大于配置，约定了一套统一的项目结构和构建规则，无需额外配置(或只需轻量配置),即可完成构建需求。
- 灵活性高，可以使用`Groovy/Kotlin` 语言编写代码实现复杂构建逻辑
- 构建速度快

**不同项目构建工具对比**

| 自动化构建工具对比 |       Ant       |    Maven    |          Gradle          |
| :----------------: | :-------------: | :---------: | :----------------------: |
|      构建性能      |      最高       |    最低     |           居中           |
|        仓库        | 开发者自己处理  |  maven仓库  |     支持多种远程仓库     |
|      依赖管理      |     ivy管理     | GAV坐标管理 |       GNV坐标管理        |
|      插件支持      |    实现方便     |  实现较难   |         实现方便         |
|  遵循特定目录结构  |       无        |    遵循     |         同maven          |
|      配置文件      | xml文件最为繁琐 |   xml文件   | 代码脚本，便于写业务逻辑 |
|       侧重点       |  小型项目构建   | 项目包管理  |       大型项目构建       |
|      目前地位      |    使用较少     |  目前主流   |   未来趋势(spring家族)   |

- Maven更擅长包管理
- Gradle更擅长大型项目构建，构建速度快，且支持编写脚本进行项目构建

## 安装

- Gradle运行依赖于JDK，且不同版本对JDK版本要求不同

```http
官网:https://gradle.org/
```

1. Gradle可以直接通过压缩包解压安装
2. 安装后配置`GRADLE_HOME`环境变量与path路径

### 配置仓库

> Gradle本身不具有本地仓库与远程仓库，但默认支持Maven的仓库，可以直接使用Maven仓库文件夹或者配置其他文件夹作为仓库

1. 配置`GRADLE_USER_HOME`环境变量，变量值为本地仓库文件夹

## 相关命令

- 检查gradle版本

  ```
  gradle -v
  gradle --version
  ```

## Gradle项目结构

> Gradle遵循约定打于配置，规定了一套统一的项目结构，遵循规定，可以实现轻量化配置甚至无配置构建项目

<img src=".\图片\image-20251009185205838.png" width="500px"/>

**`build.gradle`**

> Gradle 构建脚本,每一个gradle项目都有自己的构建脚本，它定义了该项目的构建逻辑。

**配置项**

- 项目使用的 **插件**
- 依赖管理（`dependencies`）
- 仓库（`repositories`）
- 自定义任务（`task`）
- 编译、打包、发布等步骤

------

- **项目中必须有`build.gradle`，否则报错**
- 如果指定了初始化脚本，则构建脚本会在初始化脚本之后执行

# 创建Gradle项目

**方式一**

1. 创建项目文件夹，在文件夹中调用`gradle init`命令

   ```
   gradle init
   ```

2. 在命令行中配置项目相关信息后即可创建完成

# 常用指令

**clean**

```
gradle clean
```

- 清空build目录

**classed**

```
gradle classed
```

- 编译业务代码与配置文件

**test**

```
gradle test
```

- 编译测试代码，生成测试报告

**build**

```
gradle build 
```

- 构建项目,构建前默认执行`classed`,`test`
- `-x test`： 跳过测试直接构建

# Gradle初始化脚本

> 在Gradle安装目录下，有一个`init.d`文件夹，可以在这个文件夹中添加一些以`.gradle`结尾的初始化脚本，这些脚本会在`gradle`在进行`build`前执行

**初始化仓库**

> 可以在初始化脚本中初始化本地仓库与远程仓库的地址

<iframe src=".\resource\init.gradle"><iframe/>

- `repositories `项为项目依赖的下载地址，`buildscript`项为项目构建插件的下载地址
- `mavenLocal()`方法配置Maven本地仓库作为gradle的远程仓库，因此依赖于Maven本地仓库，它首先会去当前用户目录下的`.m2.setting.xml`文件中寻找本地仓库地址，如果不存在则寻找`M2_HOME`环境变量，应指向所使用Maven的安装目录，根据环境变量地址寻找`conf.settings.xml`文件；如果仍不存在，则将当前用户目录下`.m2.repository`目录作为本地仓库
- gradle会将从远程仓库下载的依赖存储在自己的缓存目录中,默认在当前用户目录的`.gradle/caches`目录，如果配置了`GRADLE_USER_HOME`环境变量，则缓存在`GRADLE_USR_HOME/caches`中，caches中管理jar包的方式与Maven不同

## 启用初始化脚本

**方式一**

> 命令行中执行命令指定初始化脚本

```
gradle --init-script yourdir/init.gradle -q taskName
```

- 多次输入此命令来指定多个init文件

**方式二**

> 将初始化脚本文件放入当前用户目录下的`.gradle`文件夹中

**方式三**

> 将初始化脚本放入当前用户目录下的`.gradle/init.d`中

**方式四**

> 将初始化脚本放入gradle安装目录下的`init.d`目录中

- 如果使用多种方式启用初始化脚本，gradle会按上面的1-4序号依次执行这些文件，如果给定目录下存在多个init脚本，会按拼音a-z顺序执行这些脚本，每个init脚本都存在一个对应的gradle实例,你在这个文件中调用的所有方法和属性，都会委托给这个gradle实例，每个init脚本都实现了Script接口。

# 包装器

> 即`Gradle Wrapper`,本质是对Gradle的一层包装，用以解决开发中不同项目可能需要使用不同版本的Gradle问题

**有了Gradle Wrapper后，本地可以不配置Gradle，可以直接使用Gradle项目中自带的Wrapper进行操作**

- 标准Gradle项目结构中，包装器位于`gradle\wrapper`目录中，标准目录还存在`gradlew`与`gradlew.bat`命令文件，这两个文件依赖于`wrapper`目录中配置的gradle执行命令
- `gradlew`与·`gradlew.bat`的使用方式与`gradle`命令完全一致，**只是通过项目中的包装器完成命令**

**修改包装器的Gradle版本**

> 可以通过执行`gradle wrapper`命令时，通过下列选项指定gradle版本

|                            |                               |
| :------------------------: | :---------------------------: |
|     `--gradle-version`     |      指定所需Gradle版本       |
| `--gradle-distrbution-url` | 指定下载Gradle发行版的URL地址 |

- `gradle wrapper`只是在修改`gradle/wrapper/gradle-wrapper.properties`的配置信息，在第一次执行`gradlew`命令后才会真正下载Gradle

- 在下载Gradle后，会将其缓存到由`gradle/wrapper/gradle-wrapper.properties`配置的目录中，之后就会通过新的Gradle执行`gradlew`命令

- `gradle-wrapper.properties`配置项说明

  |       字段名       |                        说明                        |
  | :----------------: | :------------------------------------------------: |
  | `distributionBase` |        下载的Gradle压缩包解压后存储的主目录        |
  | `distributionPath` | 相对于distributionBase的解压后的Gradle压缩包的路径 |
  |   `zipStoreBase`   |    同distributionBase，只不过是存放zip压缩包的     |
  |   `zipStorePath`   |    同distributionPath，只不过是存放zip压缩包的     |
  | `distributionUrl`  |            Gradle发行版压缩包的下载地址            |

  - 如果没有配置过 `GRALE_USER_HOME `环境变量,默认存放在当前用户目录下的.gradle 文件夹中。

# Groovy

> 可以视为Java的一种脚本化改良版,依赖JVM运行，可以很好的与Java代码及相关库进行交互操作。是一种成熟的面向对象语言，既支持面向对象编程，也可作为纯粹的脚本语言。大多数有效的Java代码可以转换为有效的Groovy代码

**特点**

- 功能强大，例如提供了动态类型转换、闭包和元编程（metaprogramming）支持
- 支持函数式编程，不需要 main 函数
- 默认导入常用的包
- 类不支持 default 作用域,且默认作用域为 public。
- Groovy 中基本类型也是对象，可以直接调用对象的方法。
- 支持 DSL（Domain Specific Languages 领域特定语言）和其它简洁的语法，让代码变得易于阅读和维护
- Groovy 是基于 Java 语言的，所以完全兼容 Java 语法,所以对于 java 程序员学习成本较低。